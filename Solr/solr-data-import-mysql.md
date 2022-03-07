## How to : Guide on importing data from MySQL into Apache Solr


### Data Setup in MySQL

#### DDL

```sql
CREATE TABLE BOOK_CATALOUGUE (
  BOOK_ID INT AUTO_INCREMENT,
  BOOK_TITLE VARCHAR(100),
  BOOK_DESC VARCHAR(250),
  AUTHOR VARCHAR(150),
  TAGS VARCHAR(150),
  EDITION INT,
  PAGES INT,
  LANGUAGE VARCHAR(10) DEFAULT 'ENGLISH',
  UPDATED_AT timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  PRIMARY KEY (BOOK_ID)
);

CREATE TABLE BOOK_REVIEWS (
  BOOK_ID INT,
  REVIEW VARCHAR(300),
  FOREIGN KEY (BOOK_ID) REFERENCES BOOK_CATALOUGUE(BOOK_ID)
);
```

#### DML

```sql
INSERT INTO BOOK_CATALOUGUE (BOOK_TITLE, BOOK_DESC, AUTHOR, TAGS, EDITION, PAGES) VALUES (
'Java the Complete Reference', 
'Comprehensive book for UG Computer Science Engineering students. The book comprises of chapters on the Java language, the Java library etc.',
'Herbert Schildt',
'java,programming,language',
2012,
685);

INSERT INTO BOOK_CATALOUGUE (BOOK_TITLE, BOOK_DESC, AUTHOR, TAGS, EDITION, PAGES) VALUES (
'Java Network Programming', 
'Developing Networked Applications with Java',
'Harold Elliotte Rusty',
'java,programming,network',
2014,
423);

INSERT INTO BOOK_CATALOUGUE (BOOK_TITLE, BOOK_DESC, AUTHOR, TAGS, EDITION, PAGES) VALUES (
'Selenium WebDriver Practical Guide', 
'Comprehensive book for QA professionals. It helps learn designs of Selenium WebDriver, handling cookies, browser navigations, taking screenshots & managing timeouts',
'Avasarala Satya',
'java,testing,selenium,framework',
2019,
200);

INSERT INTO BOOK_CATALOUGUE (BOOK_TITLE, BOOK_DESC, AUTHOR, TAGS, EDITION, PAGES) VALUES (
'Shell Scripting Recipes', 
'Comprehensive and example-driven, for faster completion of administration tasks. Scripts are POSIX-compliant; supported by all mainstream shells',
'Chris Johnson',
'unix,linux,shell,scripting',
2017,
448);

INSERT INTO BOOK_CATALOUGUE (BOOK_TITLE, BOOK_DESC, AUTHOR, TAGS, EDITION, PAGES) VALUES (
'Java the Good Parts', 
'Java the Good Parts',
'Waldo Jim',
'java,programming,language',
2019,
212);

INSERT INTO BOOK_CATALOUGUE (BOOK_TITLE, BOOK_DESC, AUTHOR, TAGS, EDITION, PAGES) VALUES (
'Spring Web Services 2 Cookbook', 
'This is a cookbook full of recipes with the essential code explained clearly and comprehensively. Each chapter is neatly compartmentalized with focused recipes which are perfectly organized for easy reference and understanding',
'Sattari Hamidreza',
'java,spring,framework',
2018,
322);

INSERT INTO BOOK_CATALOUGUE (BOOK_TITLE, BOOK_DESC, AUTHOR, TAGS, EDITION, PAGES) VALUES (
'Mastering Node.js', 
'This book is great for anyone who already knows JavaScript and who wants to start creating 3D graphics that run in any browser',
'Pasquali Sandro',
'node,javascript,nodejs',
2017,
346);

INSERT INTO BOOK_CATALOUGUE (BOOK_TITLE, BOOK_DESC, AUTHOR, TAGS, EDITION, PAGES) VALUES (
'JavaScript Testing Beginners Guide', 
'This book is organized such that only the most essential information is provided to you in each chapter so as to maximize your learning',
'Eugene Liang Yuxian',
'testing,javascript',
2014,
285);

INSERT INTO BOOK_CATALOUGUE (BOOK_TITLE, BOOK_DESC, AUTHOR, TAGS, EDITION, PAGES) VALUES (
'Hands-On Full Stack Development with Spring Boot 2.0 and React', 
'This book is organized such that only the most essential information is provided to you in each chapter so as to maximize your learning',
'Hinkula Juha',
'springboot,spring,react,fullstack,full stack',
2019,
302);

INSERT INTO BOOK_CATALOUGUE (BOOK_TITLE, BOOK_DESC, AUTHOR, TAGS, EDITION, PAGES) VALUES (
'JQuery in Action', 
'jQuery In Action is a book that provides an introduction and guidelines to use jQuery in order to reduce the workload and increase the efficiency while developing JavaScript or web-based applications.',
'Bibeault Bear',
'javascript,jquery',
2014,
484);

INSERT INTO BOOK_CATALOUGUE (BOOK_TITLE, BOOK_DESC, AUTHOR, TAGS, EDITION, PAGES) VALUES (
'Learn Android App Development', 
'Learn Android App Development',
'Jackson Wallace',
'mobile,android,app development',
2012,
536);

INSERT INTO BOOK_REVIEWS VALUES (1, 'Extraordinary book to start with Java');
INSERT INTO BOOK_REVIEWS VALUES (1, 'Beginners guide to start with Java');
INSERT INTO BOOK_REVIEWS VALUES (1, 'The first book I read about Java');
INSERT INTO BOOK_REVIEWS VALUES (2, 'Best book to learn networking with Java');
INSERT INTO BOOK_REVIEWS VALUES (3, 'Believe me, this book helped me land in a job');
INSERT INTO BOOK_REVIEWS VALUES (4, 'Always remembered shell scripting as tough but not any more');
INSERT INTO BOOK_REVIEWS VALUES (6, 'Must read for Professional Spring Developers');
INSERT INTO BOOK_REVIEWS VALUES (10, 'JQuery is so easy after this book');
INSERT INTO BOOK_REVIEWS VALUES (10, 'Easy to learn. Explained with examples');
```

#### managed-schema

```xml
<field name="id" type="string" indexed="true" stored="true" required="true" multiValued="false" />
<!-- docValues are enabled by default for long type so we don't need to index the version field  -->
<field name="_version_" type="plong" indexed="false" stored="false"/>
<field name="_root_" type="string" indexed="true" stored="false" docValues="false" />
<!-- <field name="_text_" type="text_general" indexed="true" stored="false" multiValued="true"/> -->

<!-- <field name="book_id" type="pint" indexed="true" stored="true" required="true" multiValued="false" /> -->

<field name="title" type="text_en" indexed="true" stored="true" required="true" multiValued="false" />

<field name="desc" type="text_en" indexed="false" stored="true" required="true" multiValued="false" />

<field name="author" type="text_general" indexed="true" stored="true" required="true" multiValued="false" />

<field name="tags" type="string" indexed="true" stored="true" required="true" multiValued="true" />

<field name="edition" type="pint" indexed="false" stored="true" required="true" multiValued="false" />

<field name="pages" type="pint" indexed="false" stored="true" required="true" multiValued="false" />

<field name="language" type="lowercase" indexed="false" stored="true" required="true" multiValued="false" />

<field name="review" type="string" indexed="false" stored="true" required="false" multiValued="true" />
```

#### solrconfig.xml

```xml
<lib dir="${solr.install.dir:../../../..}/dist/" regex="solr-dataimporthandler-.*\.jar" />
<lib dir="${solr.install.dir:../../../..}/dist/" regex="mysql-connector-java-8.0.21.jar" />


<requestHandler name="/dataimport" class="org.apache.solr.handler.dataimport.DataImportHandler">
  <lst name="defaults">
    <str name="config">data-config.xml</str>
    </lst>
</requestHandler>
```

#### data-config.xml

```xml
<dataConfig>
  <dataSource type="JdbcDataSource" 
              driver="com.mysql.jdbc.Driver"
              url="jdbc:mysql://localhost:3306/book_bazzar" 
              user="root" 
              password="rootroot"/>
  <document>
    <entity name="Book"  
      pk="BOOK_ID"
      query="select BOOK_ID,BOOK_TITLE,BOOK_DESC,AUTHOR,TAGS,EDITION,PAGES,LANGUAGE from BOOK_CATALOUGUE"
      deltaImportQuery="SELECT BOOK_ID,BOOK_TITLE,BOOK_DESC,AUTHOR,TAGS,EDITION,PAGES,LANGUAGE from BOOK_CATALOUGUE WHERE BOOK_ID='${dih.delta.id}'"
      deltaQuery="SELECT BOOK_ID FROM BOOK_CATALOUGUE WHERE UPDATED_AT > '${dih.last_index_time}'"
      transformer="RegexTransformer"
      >
       <field column="BOOK_ID" name="id"/>
       <field column="BOOK_TITLE" name="title"/> 
       <field column="BOOK_DESC" name="desc"/> 
       <field column="AUTHOR" name="author"/>
       <field column="TAGS" name="tags" splitBy=","/>     
       <field column="EDITION" name="edition"/> 
       <field column="PAGES" name="pages"/>
       <field column="LANGUAGE" name="language"/>

       <entity name="Book_Reviews" pk="BOOK_ID" query="SELECT * FROM BOOK_REVIEWS WHERE BOOK_ID='${Book.BOOK_ID}'">
          <field column="REVIEW" name="review" />  
        </entity>
    </entity>
  </document>
</dataConfig>
```
