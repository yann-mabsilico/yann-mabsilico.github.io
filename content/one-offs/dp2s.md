---
layout: default
title: docker
---

{: .title}
postgresql - Installation

* TOC
{: toc}

# Root

    apt-get install postgresql python3-psycopg2

# Non-root, but `postgres` user
<https://www.postgresqltutorial.com/install-postgresql-linux/>
## From linux shell

	/etc/init.d/postgresql start
	psql

## From psql shell
	create database kikoo;
	# Connecting to `kikoo`.
	\c kikoo
	# Listing tables.
	\dt
	# Getting info of a table.
	\d table_name

## Using `python`
<https://www.postgresqltutorial.com/postgresql-python/connect/>

