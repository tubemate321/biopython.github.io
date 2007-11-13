---
title: BioSQL
permalink: wiki/BioSQL
layout: wiki
---

[BioSQL](http://www.biosql.org/wiki/Main_Page) is a joint effort between
the [OBF](http://open-bio.org/) projects (BioPerl, BioJava etc) to
support a shared database schema for storing sequence data. In theory,
you could load a GenBank file into the database with BioPerl, then
extract this from the database as a [SeqRecord](SeqRecord "wikilink")
with featues using Biopython - and get more or less the same thing as if
you had loaded the GenBank file directly using
[SeqIO](SeqIO "wikilink").

Existing documentation for the Biopython interfaces to BioSQL cover
installing Python database adaptors and basic usage of BioSQL:

[HTML](http://biopython.org/DIST/docs/biosql/python_biosql_basic.html) |
[PDF](http://biopython.org/DIST/docs/biosql/python_biosql_basic.pdf)

I hope to use this wiki page to update the above documentation in
future.

NOTE - At the time of writing, there are a few problems with BioSQL and
Biopython 1.44 which are being tackled...

Installation
============

This is fairly complicated - partly because there are some many options.
For example, you can use a range of different SQL database packages
(we'll focus on MySQL), you can have the database on your own computer
(the assumption here) or on a separate server. Also the details will
vary depending on your operating system.

This text is based in part on the [BioSQL scheme INSTALL
instructions](http://code.open-bio.org/cgi/viewcvs.cgi/*checkout*/biosql-schema/INSTALL?rev=HEAD&cvsroot=biosql&content-type=text/plain),
which also covers alternatives to MySQL.

Installing Required Software
----------------------------

You will need to install some database software plus the associated
python library so that Biopython can "talk" to the database. In this
example we'll talk about the most common choice, MySQL. How you do this
will also depend on your operating system, for example on a Debian or
Ubuntu Linux machine try this:

`sudo apt-get install mysql-common mysql-server python-mysqldb`

It will also be important to have perl (to run some of the setup
scripts) and cvs (to get some BioSQL files). Again, on a Debian or
Ubuntu Linux machine try this:

`sudo apt-get install perl cvs`

You may find perl is already installed.

Downloading the BioSQL Schema
-----------------------------

One the software is installed, your next task is to setup a database and
import the BioSQL scheme (i.e. setup the relevant tables within the
database). If you have CVS installed, then on Linux you can download the
latest schema like this (password is 'cvs').

`cd ~`  
`mkdir repository`  
`cd repository`  
`cvs -d :pserver:cvs@code.open-bio.org:/home/repository/biopython checkout biosql`  
`cvs -d :pserver:cvs@code.open-bio.org:/home/repository/biosql checkout biosql-schema`  
`cd biosql-schema/sql`

If you don't want to use CVS, then download the files via the [View CVS
web
interface](http://cvs.open-bio.org/cgi-bin/viewcvs/viewcvs.cgi/?cvsroot=biosql).
Click the Download tarball link to get a tar.gz file containing the all
the current CVS file, and then unzip that. Or, navigate to the relevant
file for your database and download just that, e.g.
[biosql-schema/sql/biosqldb-mysql.sql](http://cvs.open-bio.org/cgi-bin/viewcvs/viewcvs.cgi/biosql-schema/sql/biosqldb-mysql.sql?cvsroot=biosql)
for MySQL.

Creating the empty database
---------------------------

Assuming you are using MySQL, the following command line should create a
new database on your own computer called *bioseqdb*, belonging to the
*root* user account:

`mysqladmin -u root create bioseqdb`

We can then tell MySQL to load the BioSQL scheme we downloaded above
(file ~/repository/biosql-schema/sql/biosqldb-mysql.sql if you
downloaded the files where we suggested).

`cd ~/repository/biosql-schema/sql`  
`mysql -u root bioseqdb < biosqldb-mysql.sql`

You can have a quick play using the mysql command line tool, for
example:

`mysql --user=root bioseqdb -e "select * from bioentry;"`

This should return no rows as the table is empty.

NCBI Taxonomy
-------------

Before you start trying to load sequences into the database, it is a
good idea to load the NCBI taxonomy database using the
scripts/load\_ncbi\_taxonomy.pl script in the BioSQL package.

The script should be able to download the files it needs from the [NCBI
taxonomy FTP site](ftp://ftp.ncbi.nih.gov/pub/taxonomy/) automatically:

`cd ~/repository/biosql-schema/scripts`  
`./load_ncbi_taxonomy.pl --dbname bioseqdb --driver mysql --dbuser root --download true`

There is about 10MB to fetch, so it can take a little while (and doesn't
give any feedback while this happens). If you are worried, open a file
browser window and check to see it is downloading a file called
taxdump.tar.gz to the taxdata subdirectory.

You should see this output at the command prompt - be warned that some
of these steps do take a while (especially *rebuilding nested set
left/right values*):

`Loading NCBI taxon database in taxdata:`  
`        ... retrieving all taxon nodes in the database`  
`        ... reading in taxon nodes from nodes.dmp`  
`        ... insert / update / delete taxon nodes`  
`        ... (committing nodes)`  
`        ... rebuilding nested set left/right values`  
`        ... reading in taxon names from names.dmp`  
`        ... deleting old taxon names`  
`        ... inserting new taxon names`  
`        ... cleaning up`  
`Done.`

This might be a good point for a tea break - I didn't time this but it
was over ten minutes.

Running the unit test
---------------------

Because there are so many ways you could have setup your BioSQL
database, you have to tell the unit test a few bits of information. You
need to find the Biopythin file under Tests/test\_BioSQL.py and fill in
the following fields - based on the setup described above:

`DBDRIVER = 'MySQLdb'`  
`DBTYPE = 'mysql'`

and a little lower down,

`DBHOST = 'localhost'`  
`DBUSER = 'root'`  
`DBPASSWD = ''`  
`TESTDB = 'biosql_test'`

You can then run the unit test as normal, e.g.

`python runtests test_BioSQL`

Right now there are a few rough edges and the test will fail :(

Loading Sequences into the database
===================================

When loading sequences into a BioSQL database with Biopython we have to
provide annotated [SeqRecord](SeqRecord "wikilink") objects. This gives
us another excuse to use the [SeqIO](SeqIO "wikilink") module! A quick
recap on reading in sequences as SeqRecords, based on one of the orchid
examples in the Biopython Tutorial:

``` python
from Bio import GenBank
from Bio import SeqIO
handle = GenBank.download_many(['6273291', '6273290', '6273289'])
for seq_record in SeqIO.parse(handle, "genbank") :
    print seq_record.id, seq_record.description[:50] + "..."
    print "Sequence length %i," % len(seq_record.seq),
    print "from: %s" % seq_record.annotations['source']
handle.close()
```

The expected output is:

`AF191665.1 Opuntia marenae rpl16 gene; chloroplast gene for c...`  
`Sequence length 902, 3 features, from: chloroplast Opuntia marenae`  
`AF191664.1 Opuntia clavata rpl16 gene; chloroplast gene for c...`  
`Sequence length 899, 3 features, from: chloroplast Grusonia clavata`  
`AF191663.1 Opuntia bradtiana rpl16 gene; chloroplast gene for...`  
`Sequence length 899, 3 features, from: chloroplast Opuntia bradtianaa`

instea Now, instead of printing things on screen, let's add these three
records to a new (empty) *orchid* database:

``` python
from BioSQL import BioSeqDatabase
server = BioSeqDatabase.open_database(driver="MySQLdb", user="root",
                     passwd = "", host = "localhost", db="bioseqdb")
db = server.new_database("orchids", description="Just for testing")
server.adaptor.commit()
...
```

The call *server.adaptor.commit()* is a work around for [bug
2395](http://bugzilla.open-bio.org/show_bug.cgi?id=2395).

You can check this has done something at the command line:

`mysql --user=root bioseqdb -e "select * from biodatabase;"`

There should be a single row in the *biodatabase* table for our new
orchid database.

``` python
...
from Bio import GenBank
from Bio import SeqIO
handle = GenBank.download_many(['6273291', '6273290', '6273289'])
db.load(SeqIO.parse(handle, "genbank"))
server.adaptor.commit()
```

Again, the call *server.adaptor.commit()* is a work around for [bug
2395](http://bugzilla.open-bio.org/show_bug.cgi?id=2395).

The *db.load()* function should have returned the number of records
loaded (three in this example), and again have a look in the database
and you should see three new rows in several tables (including the
*bioentry* and *biosequence* tables):

`mysql --user=root bioseqdb -e "select * from bioentry;"`  
`mysql --user=root bioseqdb -e "select * from biosequence;"`

Next, we'll try and load these three records back from the database.

Extracting Sequences from the database
======================================

This continues from the previous example, where we loaded three records
into an *orchids* database:

``` python
from BioSQL import BioSeqDatabase
server = BioSeqDatabase.open_database(driver="MySQLdb", user="root",
                     passwd = "", host = "localhost", db="bioseqdb")
db = server["orchids"]
for identifiers in ['6273291', '6273290', '6273289'] :
    seq_record = db.lookup(gi=identifiers)
    print seq_record.id, seq_record.description[:50] + "..."
    print "Sequence length %i," % len(seq_record.seq)
```

Giving:

`AF191665.1 Opuntia marenae rpl16 gene; chloroplast gene for c...`  
`Sequence length 902`  
`AF191664.1 Opuntia clavata rpl16 gene; chloroplast gene for c...`  
`Sequence length 899`  
`AF191663.1 Opuntia bradtiana rpl16 gene; chloroplast gene for...`  
`Sequence length 899`

Todo - sort out the annotation.