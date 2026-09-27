# Pset 4: Web Application (Version 2)

### Due Friday Nov 21 10:59 PM NHT (New Haven Time)

## Table of Contents

- [Pset 4: Web Application (Version 2)](#pset-4-web-application-version-2)
  - [Due Friday Nov 21 10:59 PM NHT (New Haven Time)](#due-friday-nov-21-1059-pm-nht-new-haven-time)
  - [Table of Contents](#table-of-contents)
  - [Purpose](#purpose)
  - [Rules](#rules)
  - [Getting Started](#getting-started)
  - [Your Task](#your-task)
  - [The Database](#the-database)
  - [Background](#background)
  - [The Application](#the-application)
    - [Output Requirements](#output-requirements)
    - [Filtering Requirements](#filtering-requirements)
    - [The Details Page](#the-details-page)
  - [The Yale Courses API](#the-yale-courses-api)
  - [Bonus: Build `reg.sqlite` from the API](#bonus-build-regsqlite-from-the-api)
  - [Additional Requirements for 519 Students](#additional-requirements-for-519-students)
    - [Additional Features](#additional-features)
  - [A Note About Performance](#a-note-about-performance)
  - [Object-Relational Mappers](#object-relational-mappers)
  - [Specific Endpoints](#specific-endpoints)
  - [Error Handling: Bad Server](#error-handling-bad-server)
  - [Error Handling: Bad Client](#error-handling-bad-client)
    - [Invalid search parameters](#invalid-search-parameters)
    - [Invalid `crn`](#invalid-crn)
    - [Missing `crn`](#missing-crn)
    - [Aside: Error Pages](#aside-error-pages)
    - [Other invalid requests](#other-invalid-requests)
  - [Source Code Guide](#source-code-guide)
  - [Submission](#submission)
    - [Late Submissions](#late-submissions)
  - [Grading](#grading)

## Purpose

The purpose of this assignment is to help you learn or review client-side web programming.

---

## Rules

You may work with one teammate on this assignment, and we prefer that you do so.

It must be the case that either you submit all of your team’s files or your teammate submits all of your team’s files.
(It must not be the case that you submit some of your team’s files and your teammate submits some of your team’s files.)
Your `README` file and your source code files must contain your name and your teammate’s name.

---

## Getting Started

> **Note**: this section contains exactly the text on the Canvas assignment, reproduced here only for completeness of this document.
> Since you made it here, you can safely ignore this section.

1. Accept the GitHub classroom assignment.

2. The `reg.sqlite` database file is included in your GitHub repository, so no separate download or setup is required.

---

## Your Task

As with Psets 1, 2, and 3, assume you are working for Yale's Registrar's Office.
You are given a database containing data about courses and classes offered during this semester at Yale.
Your task is to compose an application that allows Yale students and other interested parties to query the database.

This assignment asks you to compose a web application.
On the server side, your application must use the Python Flask framework and the Jinja2 template engine.
If you are interested, you may use SQLAlchemy to manage your database queries, which is an Object Relational Mapper (ORM) for Python.

On the client side, your application must use HTML and JavaScript with AJAX, and may use jQuery (which we recommend).
It must also use CSS, and to keep things easy on yourself you may use Bootstrap.
Your application may not use any other client-side library or framework (such as AngularJS, Vue, React, and so forth).

---

## The Database

The database is identical to the one from Psets 1, 2, and 3.
Refer to the Pset 1 specification for the list of tables and fields in the database, and for the relationships among those tables.
The database is a SQLite database in a file named `reg.sqlite`, included in this template repository.

Department names in this database include the department code, for example `Computer Science (CPSC)`.
A course can be crosslisted under more than one subject code and course number.
Some courses have several sections, several meetings, several professors, or none.

---

## Background

The application you built in Pset 3 is flawed.
The flaw is not in your implementation; instead the flaw (intentionally) is in the specification.

The problem is that an application that conforms to that specification often has inconsistent page states.
For example, consider the following sequence of events:

* The user browses to your website and the browser displays your application's primary page
* The user types "web" into the "title" input element, but is distracted before the search runs
* Sometime later the user returns to the page

At that point the page's "title" input element contains "web" but the page displays no data.
This is an inconsistent state, and it is a problem that you will fix in this assignment.

---

## The Application

Compose a program named `runserver.py`.
When executed with `-h` as a command-line argument, the program must display the following help message describing the program's behavior:

```text
$ python runserver.py -h
usage: runserver.py [-h] port

The registrar search application

positional arguments:
  port        the port at which the server should listen

options:
  -h, --help  show this help message and exit
```

> **Note**: The specific verbiage of this help message should be the default for your version of the `argparse` module, which differs slightly between Python versions.

Your `runserver.py` must run an instance of the Flask test server on the specified port, which must in turn run your application.

> **Note**: Your application code should *not* be in your `runserver.py` program, which should do nothing but start a Flask server on the provided port number.
> The file containing your application code may be named whatever you want, but we suggest something simple such as `regapp.py`.

When a client makes an HTTP request to the server, your application must return an HTML webpage appropriate to the request.
Beyond some [specific endpoints](#specific-endpoints) mentioned below, there are several requirements your application must satisfy:

* Your application's **primary web page** (*i.e.*, the webpage returned by a request to the server's root&mdash;*e.g.* `http://yourserver:80/` if your server is listening on port 80) must contain four text input fields labeled "dept", "coursenum", "subjectcode", and "title".
  These should, respectively, allow the user to specify a department code, a course number, a subject code, and a title.
  * In contrast to the form from Pset 3, your primary page must *not* contain a submit button.
  * The input fields must be filled in with the parameters used to make the most recent query, that is, the values of the input fields as they were upon the most recent query (if a query has never been performed, those fields should be empty)
* Below the input fields, the webpage must display an HTML `table` containing no more than the first 1000 results of the most recent query (or nothing at all if no query has yet been sent)

### Output Requirements

The columns displayed in the table must be, in order, `deptname`, `subjectcode`, `coursenum`, `title`, and `crns`, for each course that matches the specified criteria, or for all courses in the database if the user specifies no criteria.
The columns must be labeled "deptname", "subjectcode", "coursenum", "title", and "crns".

* The `deptname` column contains the department name from the `departments` table.
* The `subjectcode`, `coursenum`, and `title` columns each match the corresponding field of the course.
* The `crns` column contains the `crn` of each section associated with the course, one CRN per line, sorted in increasing order.
* There is one table row per course. Repeat `deptname`, `subjectcode`, `coursenum`, and `title` on that row, and list every CRN of the course in the `crns` cell.
* The table rows must be sorted first by `deptcode` in ascending order, then by `subjectcode` in ascending order, then by `coursenum` in ascending order, and finally by `title` in ascending order.
* A user must be able to click on a `crn` to request more information about that section on a different webpage (the details page) at the url `/crn/{crn}`.
  The requirements for the details page are below.

  > **Note**: The course's internal `courseid` must not be displayed in the table.

* The results table on the primary webpage must be updated every time the user types a character in any of the four input fields

  > **Note**: You're required to use JavaScript and AJAX to accomplish this, and we recommend you use jQuery as well to keep your code concise and easy to understand.

For example, a search with dept `cpsc` and coursenum `4190` includes a row such as:

```text
Computer Science (CPSC)    CPSC    4190    Full Stack Web Programming    10859
```

A search with dept `chem` and coursenum `6000` includes one row whose `crns` cell lists every section, one CRN per line:

```text
Chemistry (CHEM)    CHEM    6000    Research Seminar    13753
                                                            13754
```

Column spacing in the browser will depend on your HTML and CSS. The examples illustrate the fields and the one-row-per-course rule.

### Filtering Requirements

The four text fields correspond to the filters from Pset 1.
An empty field does not constrain the query.
If a query has been made and every field is empty, the result is every course in the database, up to the 1000-row limit, in the sort order above.
Before any query has been made, the page shows no results table.

| Field | Meaning |
| --- | --- |
| dept | Include courses whose `deptcode` contains the supplied value. |
| coursenum | Include courses whose `coursenum` contains the supplied value. |
| subjectcode | Include courses whose `subjectcode` contains the supplied value. |
| title | Include courses whose `title` contains the supplied value. |

If several fields are non-empty, combine the filters using `AND`.
Filters must be case-insensitive: a title field containing `web` must match a title such as "Full Stack Web Programming".
Filters must preserve leading and trailing whitespace.

### The Details Page

Your application must accept requests to the endpoint `/crn/<crn>` (where `crn` is the CRN of the clicked-on section).
The webpage returned at this endpoint must show the same information, in the same sections, as `regdetails.py` from Pset 1 for that `crn`:

* A section containing a single-row HTML table with the columns `deptcode`, `deptname`, `subjectcode`, and `coursenum`
* A section with header `title`, containing the course title
* A section with header `descrip`, containing the course description, or the string `None` if there is no description
* A section with header `prereqs`, containing the course prerequisites, or the string `None` if there are no prerequisites
* An HTML table with the columns `sectionnumber`, `crn`, and `meetinginfo`
  * Include every meeting of every section of the course associated with the requested `crn`
  * Each meeting must be formatted as the meeting time, followed by ` @ `, followed by the meeting location, for example `MW 9.00-10.15 @ WTS A74`
  * Put one meeting on each line
* An HTML table of crosslistings, with the columns `subjectcode` and `coursenum`
* A section with header `professors`, containing each professor's `profname`, one per line

> **Note**: The description and prerequisites strings for some courses contain HTML.
> That HTML must be rendered according to the included elements.

* Its information must be well-formatted (*e.g.*, the headers must be within semantic HTML `<h`*`N`*`>` header elements, and the lists must be within HTML `<ul>` elements)
* Unlike Pset 3, the details page must not provide a link back to the primary webpage.
  Instead, the details page must be displayed by default in a **new** tab or window.

  > **Note**: Read about the `target="_blank"` attribute of the `a` element

* The default styling of an HTML table is quite ugly.
  Use CSS to spruce up your tables:
  * Add padding of `5px` to all sides of every cell in each table
  * Add a `1px` wide `gray` border between rows of each table
  * When the mouse hovers over a particular row of each table, change that row's background color to `lightgray`
  * You are free to style your application in any other manner that looks good to you (but nothing else is required)
  * Place your CSS into a file named `styles.css` that is loaded by your primary and details webpages, but not included directly in those pages

---

## The Yale Courses API

The pages above search `reg.sqlite`.
The application must also retrieve course data from the Yale Courses API and must be able to search that retrieved data.

Cache the retrieved API data in one of these ways:

* a global variable initialized at server startup
* a global variable initialized on the first request to the server
* one or more local files that the program reads when it receives a request

The cache must include the subjects available from the Subjects API.
A program that never queries the Subjects API, and that has no cached information about those subjects, does not meet this requirement.

Searching the API data must handle crosslistings.
It must also match a department code to the corresponding department name.
Display meetings from the API in the same `meeting time @ meeting location` form required on the details page, for every meeting of a course.

---

## Bonus: Build `reg.sqlite` from the API

This part is optional.

Write a program that retrieves data from the Yale Courses API and stores it in a relational database whose schema is identical to `reg.sqlite`.

A complete database:

* contains every course from every school and every subject returned by the API
* matches the provided `reg.sqlite` file, aside from changes in the API data since that file was built
* stores meeting times and locations, crosslistings, professors, and departments
* stores department names with the department code concatenated, in the same form as the provided database, for example `Computer Science (CPSC)`

A database that is otherwise complete, but that stores the department name without the concatenated department code, still meets the goal of this bonus.
A database that contains only some schools, for example only Yale College and the Summer Session, does not.

---

## Additional Requirements for 519 Students

If you *or your partner* are enrolled in CPSC 519, your application must satisfy all of the above requirements and you must implement some of the following features.

If at least one member of your team is enrolled in CPSC 519, you are required to implement at least one of the [Additional Features](#additional-features).

Even if you and your partner are in 419 and not 519, you are more than welcome to attempt these activities.
They will not have any bearing on your grade for this assignment, but they will provide valuable experience in adding features to a webpage that may prove useful in developing your final project.

### Additional Features

If you *or your partner* are enrolled in CPSC 519, you must implement the following additional feature.

* Large result sets are cumbersome for a user to scroll through.
  A standard technique to reduce the burden on users is **pagination**.
  A paginated results table would display only *k* items at a time, and display a row of buttons that the user could use to go to the next page or previous page (or even a particular page)
  * You are required to implement pagination for your app, which must display no more than `10` items per page, and you must provide a button to go to the next page and a button to go to the previous page
    * Optionally, also display individual page numbers so that the user can jump to a specific page
    * Optionally, also display a "first page" and "last page" button
  * The buttons must be disabled as appropriate if the user is on the first or last page of results
  * Each page of results must display the column headers
  * Results must still be limited to the first 1000 courses (*i.e.*, 100 pages)

  > **Hint**: You may have to modify somewhat the response from your server to make this feature feasible to implement.
  > In particular, if you return an HTML table from the server in response to a search request, that may be challenging to slice up into pages.
  > However, if you return a JSON list of courses, there are built-in JavaScript functions that will help you pick out chunks of the list.

If you *and your partner* are enrolled in CPSC 519, you must implement the following additional feature.

* The sorting algorithm required by your program is quite inflexible, and it would be nice if users could sort results however they want.
  * Make (part of) the column header cells clickable, and respond to a click in a column header by sorting the results in ascending order of the values in that column
    * Ties must be broken by the Pset 1 sort: `deptcode`, then `subjectcode`, then `coursenum`, then `title`
    * Sorting by the `crns` column must be done by the **first** CRN in the list
  * The reordering **must be done locally**, and clicking the header may not send a request to the server
  * Provide some visual indication in the column header when its data is in sorted order. Options include adding an asterisk, changing the color, etc.

  > **Hint**: You may have to modify somewhat the response from your server to make this feature feasible to implement.
  > In particular, if you return an HTML table from the server in response to a search request, that will be challenging to reorder.
  > However, if you return a JSON list of courses, there are built-in JavaScript functions that will help you sort things.

---

## A Note About Performance

Queries that take many seconds to return may earn some additional scrutiny that may result in some penalties for performance or code style, especially if you've received feedback on your queries from previous assignments.
Ordinary variation in response time relative to typing speed is expected, and is not something to worry about as far as your grade on this assignment is concerned.

We encourage you to explore the capabilities of an industrial-strength DBMS such as [PostgreSQL](https://www.postgresql.org/) or [MySQL](https://www.mysql.com/), both of which are free and open source for individual use.

In exploring these systems, in particular investigate how to use [Materialized Views](https://en.wikipedia.org/wiki/Materialized_view) to improve performance (for example, by materializing join tables rather than creating them from scratch on each query).
SQLite does, in a sense, [support materialization](https://www.sqlite.org/lang_with.html#mathint), but it is not the same as materialized views in an industrial-strength DBMS and in my testing actually makes our queries *slower*.

> **Note**: There are nonstandard ways around this limitation of SQLite, including by explicitly creating tables as part of server startup.
> These "cached" tables can then be used to significantly speed up your complex join queries and won't cause data integrity issues because your app doesn't change the data!
> You may find it illustrative to experiment with this, but your time is probably better spent leveraging a more sophisticated DBMS.

---

## Object-Relational Mappers

For this assignment, you may use the [SQLAlchemy ORM](https://www.sqlalchemy.org/) to aid you in your SQL queries.
You are not required to do so, but as with the additional activities for 519 students, doing so may serve as valuable experience if you choose to use SQLAlchemy in your final project or other endeavors.

---

## Specific Endpoints

There are three endpoints to which your application must respond (assume your server is listening on port 80):

1. `http://yourserver:80/` must return the primary page
2. `http://yourserver:80/search?...` must return the results of a search using the parameters in the query string
   * Requests to the `/search` endpoint need not be standalone HTML webpages. You might find it easiest to implement the rest of the assignment if this endpoint returns results structured as JSON.
   * The parameters accepted by the `/search` endpoint must be `d` (for the department code), `n` (for the course number), `s` (for the subject code), and `t` (for the title)
3. `http://yourserver:80/crn/<crn>` must return the details page populated with information about the section with the `crn` provided in the path.

Students completing the [additional 519 activities](#additional-requirements-for-519-students) may add endpoints to accomplish some of those tasks, but the exact names of those endpoints are not specified.

---

## Error Handling: Bad Server

Since you control the machine that is running both the server application and the database, there is not much we will force you to worry about for server-side errors.
There are two errors that your program must handle gracefully.

1. The server is started with a port that is not a positive integer.
   If the server is started with a command such as `$ python runserver.py notaport`, it should display a meaningful error message and exit with status code `1`.
2. The database file does not exist.
   If the server attempted to open the `reg.sqlite` file but the file does not exist (or otherwise cannot be opened), your program should display a meaningful error message and exit with status code `1`.

---

## Error Handling: Bad Client

It is an unfortunate truth about web applications of this kind that a user can enter anything they want in their request to your server!
For example, anyone can send a request to a URL such as `http://yourserver:80/crn/gobbeldygook`, despite there being no section with CRN "gobbeldygook" on which they could have clicked.
This means that we will place the same error handling requirements on your software as for Pset 3.
Specifically...

### Invalid search parameters

If the user requests a page at a URL such as `http://yourserver:80/search?foo=bar`&mdash;in which the parameters to a search request are not one of `d`, `n`, `s`, or `t`&mdash;your application must treat those parameters as not existing.
For example, in response to a request to the URL `http://yourserver:80/search?foo=bar&d=cpsc`, your server must produce results for courses whose department code contains `cpsc`.

### Invalid `crn`

If the user requests a page at a URL such as `http://yourserver:80/crn/gobbeldygook`&mdash;in which the CRN does not appear in the database&mdash;the server must return an appropriate error page as HTML that displays, at a minimum, text that reads "Error: no section with crn gobbeldygook exists". (The "gobbeldygook" part should, of course, be replaced with the actual nonexistent CRN that was queried.)
This page must be returned with status code 404 (not found).

Your application must respond with this error page for *any* missing CRN, including numeric and non-numeric CRNs.

### Missing `crn`

If the user requests a page at a URL such as `http://yourserver:80/crn` (that is, missing a `crn`), the application must return an appropriate error page as HTML that displays, at a minimum, text that reads "Error: missing crn." and status code 404 (not found).

### Aside: Error Pages

Your error page must have some required content, but it should not live at a special URL.
That is, your application should behave in a similar manner as Google when an invalid page is requested, such as `google.com/notreal`.
The page displayed should clearly indicate an error occurred, but the URL in the browser should remain `google.com/notreal`: it *should not* be some other URL, such as `google.com/404`.

### Other invalid requests

There are many other requests that the user could send that are "wrong", such as:

* `http://yourserver:notyourport/`
* `http://yourserver:80/notyourapp`
* `http://yourserver:80/search?not/well/formed/url`

The only requirement placed on your server in these cases (and similar ones) is that it does not crash when queried with such requests; that is, if the user sends a valid request immediately after an invalid one, the valid request must get the correct result.

---

## Source Code Guide

Here are the **requirements** for the source code of your solution.

* The `runserver.py` program must start a Flask server for your application on the port provided as a command-line argument, listening on all IP addresses
* Your application program must communicate with a SQLite database in a file named `reg.sqlite`, organized as described above.
  * If you explore other DBMSes, you may communicate with databases named other things or that are not SQLite databases, but you must document installation instructions in your submitted `README` file so the graders know what is going on.
* Your application program must use SQL prepared statements for every database query.
  (This protects the database against SQL injection attacks.)
  * If you use SQLAlchemy, that ORM automatically performs statement preparation and there is nothing special you need to do for this requirement.
* Use cookies to keep track of the application's state
* Your webpage must use JavaScript to update only part of the webpage in response to the user typing in the search boxes.

Here are some **recommendations** for the source code of your solution.

* Reuse code from your solution to Pset 3 in this assignment.
* Modularize your application program so that database communication code is cleanly separated from response production code.
  * Use HTML templates to keep your response production code as clear as possible
  * Structure your code according to MVC design principles (the HTML templates are your Views)
* You may use external dependencies, but we advise you not to (with the obvious exception of `flask`).
  This assignment is designed such that everything can be accomplished without too much pain using only packages from the Python standard library.

---

## Submission

Replace the provided `README.md` file (which contains this assignment specification) with your own `README.md` file that conforms to the following requirements.

1. Leave the first line of the file alone (it is the assignment title).

2. Thereafter your `README.md` file must contain:
   * Your name and Yale netid and your teammate’s name and Yale netid (if you worked with a partner)
     * Also indicate here whether you and your teammate are enrolled in 419 or 519
   * A paragraph describing your contribution, and another paragraph describing your teammate’s contribution.
     Please be thorough; we are looking for two substantial paragraphs, not a sentence or two.
   * A description of whatever help (if any) you received from other people while doing the assignment.
   * A description of the sources of information that you used while doing the assignment, that are not direct help from other people.
   * An indication of how much time you spent doing the assignment, rounded to the nearest hour.
   * Your assessment of the assignment:
     * Did it help you to learn?
     * What did it help you to learn?
     * Do you have any suggestions for improvement? *Etc.*
   * (Optionally) Any information that will help us to grade your work in the most favorable light.
     * In particular, describe all known bugs and explain why any `pylint` style warnings you received are unavoidable or why you know better than `pylint` (a convincing argument might negate some `pylint` style penalties you may accrue).
     * You should also describe any installation instructions here if you use a non-SQLite DBMS, including instructions on how to retrieve the data

Your `README.md` file must be a plain text file.
**Do not** create your `README.md` file using Microsoft Word or any other word processor, although it may be formatted using [markdown](https://www.markdownguide.org/), like this provided `README.md` file.

Package your assignment files by [creating a release](https://docs.github.com/en/repositories/releasing-projects-on-github/managing-releases-in-a-repository#creating-a-release) on GitHub in your assignment repository. There must be at least two files with the following (exact) names in that repository when you submit it:

* `README.md`
* `runserver.py`

Ensure that any additional files needed by your program (such as other Python modules, HTML templates, or CSS files) are in the repository snapshot (*i.e.*, commit) captured by the release.

> **Note**: If you have installed external packages, you must also include a file named `requirements.txt` containing the dependencies of your project.
> It can be created from your virtual environment by running the following command:
>
> ```text
> $ pip freeze -r requirements.txt
> ```
>
> Failure to include a `requirements.txt` file if you use third-party packages will result in an automatic 5% penalty and a request that you submit an appropriate `requirements.txt` file to the graders.

Submit your solution to Canvas (in the assignment named "Pset 4: Web Version 2") as [a link to that release](https://docs.github.com/en/repositories/releasing-projects-on-github/linking-to-releases).

As noted above in the [Rules](#rules) section, it must be the case that either you submit all of your team’s files or your teammate submits all of your team’s files.
(It must not be the case that you submit some of your team’s files and your teammate submits some of your team’s files.)
You and your team may submit multiple times; we will grade the latest files that you submit before the deadline unless a particular version is requested as the canonical version.

**Please follow the directions on what to submit and how.**
It will be a big help to us if you get the filenames right and submit exactly what’s asked for.
Thanks.

### Late Submissions

The deadline for this assignment is **10:59 PM NHT (New Haven Time) on Friday November 21, 2025**.
There is a strict 60 minute grace period beyond the deadline, to be used in case of technical or administrative difficulties, and not for putting final touches on your solution.

Late submissions will receive a 5% deduction for every 12-hour period (or part thereof) after the deadline.
After 48 hours, the Canvas assignment will close and submissions after that time will not receive any credit.

Except for the 48-hour cutoff, late penalties will be assessed based on the timestamp of the release, not the submission time.
Submissions after 48 hours are not accepted regardless of the timestamp on the release.

---

## Grading

Your grade will be based upon:

* **Correctness**, that is, the correctness of your programs as specified by this document.
* **Style**, that is, the quality of your program style.
  This includes not only style as qualitatively assessed by the graders (including modularity, cleanliness, and performance) but also style as reported by the `pylint` tool, using the default settings, and when executed via the command `python -m pylint **/*.py`.

Part of your grade will be based upon the quality of your program style as reported by `pylint`.
Your grader will start with the 10-point score reported by pylint.
Your *pylint style grade* is your pylint score rounded **up** to the nearest integer (minimum 0).

If your code fails the tests on some particular functionality, your grader will inspect your code manually to try to assign partial credit for that functionality.
Partial credit will be given only if there is an *obvious* "quick fix" (*e.g.*, you have accidentally changed the name of the database file and your solution points to a file with a name that does not match the grader's copy of the database); if no such quick fix exists then no partial credit for that feature will be given.

---

Original copyright &copy; 2021 by Robert M. Dondero, Jr.

This version &copy; 2026 by Alan Weide
