---
title: SHOW INDEX
summary: The SHOW INDEX statement returns index information for a table or database.
toc: true
docs_area: reference.sql
---

The `SHOW INDEX` [statement](sql-statements.html) returns index information for a table or database.


## Required privileges

The user must have any [privilege](authorization.html#assign-privileges) on the target table or database.

## Aliases

In CockroachDB, the following are aliases for `SHOW INDEX`:

- `SHOW INDEXES`
- `SHOW KEYS`

## Synopsis

<div>
{% remote_include https://raw.githubusercontent.com/cockroachdb/generated-diagrams/release-21.2/grammar_svg/show_indexes.html %}
</div>

## Parameters

Parameter | Description
----------|------------
`table_name` | The name of the table for which you want to show indexes.
`database_name` | The name of the database for which you want to show indexes.

## Response

The following fields are returned for each column in each index.

Field | Description
----------|------------
`table_name` | The name of the table.
`index_name` | The name of the index.
`non_unique` | Whether or not values in the indexed column are unique. Possible values: `true` or `false`.
`seq_in_index` | The position of the column in the index, starting with 1.
`column_name` | The indexed column.
`direction` | How the column is sorted in the index. Possible values: `ASC` or `DESC` for indexed columns; `N/A` for stored columns.
`storing` | Whether or not the `STORING` clause was used to index the column during [index creation](create-index.html). Possible values: `true` or `false`.
`implicit` | Whether or not the column is part of the index despite not being explicitly included during [index creation](create-index.html). Possible values: `true` or `false`<br><br>[Primary key](primary-key.html) columns are the only columns implicitly included in secondary indexes. The inclusion of primary key columns improves performance when retrieving columns not in the index.

## Example

{% include {{page.version.version}}/sql/movr-statements.md %}

### Show indexes for a table

{% include copy-clipboard.html %}
~~~ sql
> CREATE INDEX ON users (name);
~~~

{% include copy-clipboard.html %}
~~~ sql
> SHOW INDEX FROM users;
~~~

~~~
  table_name |   index_name   | non_unique | seq_in_index | column_name | direction | storing | implicit
-------------+----------------+------------+--------------+-------------+-----------+---------+-----------
  users      | primary        |   false    |            1 | city        | ASC       |  false  |  false
  users      | primary        |   false    |            2 | id          | ASC       |  false  |  false
  users      | primary        |   false    |            3 | name        | N/A       |  true   |  false
  users      | primary        |   false    |            4 | address     | N/A       |  true   |  false
  users      | primary        |   false    |            5 | credit_card | N/A       |  true   |  false
  users      | users_name_idx |    true    |            1 | name        | ASC       |  false  |  false
  users      | users_name_idx |    true    |            2 | city        | ASC       |  false  |   true
  users      | users_name_idx |    true    |            3 | id          | ASC       |  false  |   true
(8 rows)
~~~

### Show indexes for a database

{% include copy-clipboard.html %}
~~~ sql
> SHOW INDEXES FROM DATABASE movr;
~~~

~~~
          table_name         |                  index_name                   | non_unique | seq_in_index |   column_name    | direction | storing | implicit
-----------------------------+-----------------------------------------------+------------+--------------+------------------+-----------+---------+-----------
  users                      | primary                                       |   false    |            1 | city             | ASC       |  false  |  false
  users                      | primary                                       |   false    |            2 | id               | ASC       |  false  |  false
  users                      | primary                                       |   false    |            3 | name             | N/A       |  true   |  false
  users                      | primary                                       |   false    |            4 | address          | N/A       |  true   |  false
  users                      | primary                                       |   false    |            5 | credit_card      | N/A       |  true   |  false
  users                      | users_name_idx                                |    true    |            1 | name             | ASC       |  false  |  false
  users                      | users_name_idx                                |    true    |            2 | city             | ASC       |  false  |   true
  users                      | users_name_idx                                |    true    |            3 | id               | ASC       |  false  |   true
  vehicles                   | primary                                       |   false    |            1 | city             | ASC       |  false  |  false
  vehicles                   | primary                                       |   false    |            2 | id               | ASC       |  false  |  false
  vehicles                   | primary                                       |   false    |            3 | type             | N/A       |  true   |  false
  vehicles                   | primary                                       |   false    |            4 | owner_id         | N/A       |  true   |  false
  vehicles                   | primary                                       |   false    |            5 | creation_time    | N/A       |  true   |  false
  vehicles                   | primary                                       |   false    |            6 | status           | N/A       |  true   |  false
  vehicles                   | primary                                       |   false    |            7 | current_location | N/A       |  true   |  false
  vehicles                   | primary                                       |   false    |            8 | ext              | N/A       |  true   |  false
  vehicles                   | vehicles_auto_index_fk_city_ref_users         |    true    |            1 | city             | ASC       |  false  |  false
  vehicles                   | vehicles_auto_index_fk_city_ref_users         |    true    |            2 | owner_id         | ASC       |  false  |  false
  vehicles                   | vehicles_auto_index_fk_city_ref_users         |    true    |            3 | id               | ASC       |  false  |   true
  rides                      | primary                                       |   false    |            1 | city             | ASC       |  false  |  false
  rides                      | primary                                       |   false    |            2 | id               | ASC       |  false  |  false
  rides                      | primary                                       |   false    |            3 | vehicle_city     | N/A       |  true   |  false
  rides                      | primary                                       |   false    |            4 | rider_id         | N/A       |  true   |  false
  rides                      | primary                                       |   false    |            5 | vehicle_id       | N/A       |  true   |  false
  rides                      | primary                                       |   false    |            6 | start_address    | N/A       |  true   |  false
  rides                      | primary                                       |   false    |            7 | end_address      | N/A       |  true   |  false
  rides                      | primary                                       |   false    |            8 | start_time       | N/A       |  true   |  false
  rides                      | primary                                       |   false    |            9 | end_time         | N/A       |  true   |  false
  rides                      | primary                                       |   false    |           10 | revenue          | N/A       |  true   |  false
  rides                      | rides_auto_index_fk_city_ref_users            |    true    |            1 | city             | ASC       |  false  |  false
  rides                      | rides_auto_index_fk_city_ref_users            |    true    |            2 | rider_id         | ASC       |  false  |  false
  rides                      | rides_auto_index_fk_city_ref_users            |    true    |            3 | id               | ASC       |  false  |   true
  rides                      | rides_auto_index_fk_vehicle_city_ref_vehicles |    true    |            1 | vehicle_city     | ASC       |  false  |  false
  rides                      | rides_auto_index_fk_vehicle_city_ref_vehicles |    true    |            2 | vehicle_id       | ASC       |  false  |  false
  rides                      | rides_auto_index_fk_vehicle_city_ref_vehicles |    true    |            3 | city             | ASC       |  false  |   true
  rides                      | rides_auto_index_fk_vehicle_city_ref_vehicles |    true    |            4 | id               | ASC       |  false  |   true
  vehicle_location_histories | primary                                       |   false    |            1 | city             | ASC       |  false  |  false
  vehicle_location_histories | primary                                       |   false    |            2 | ride_id          | ASC       |  false  |  false
  vehicle_location_histories | primary                                       |   false    |            3 | timestamp        | ASC       |  false  |  false
  vehicle_location_histories | primary                                       |   false    |            4 | lat              | N/A       |  true   |  false
  vehicle_location_histories | primary                                       |   false    |            5 | long             | N/A       |  true   |  false
  promo_codes                | primary                                       |   false    |            1 | code             | ASC       |  false  |  false
  promo_codes                | primary                                       |   false    |            2 | description      | N/A       |  true   |  false
  promo_codes                | primary                                       |   false    |            3 | creation_time    | N/A       |  true   |  false
  promo_codes                | primary                                       |   false    |            4 | expiration_time  | N/A       |  true   |  false
  promo_codes                | primary                                       |   false    |            5 | rules            | N/A       |  true   |  false
  user_promo_codes           | primary                                       |   false    |            1 | city             | ASC       |  false  |  false
  user_promo_codes           | primary                                       |   false    |            2 | user_id          | ASC       |  false  |  false
  user_promo_codes           | primary                                       |   false    |            3 | code             | ASC       |  false  |  false
  user_promo_codes           | primary                                       |   false    |            4 | timestamp        | N/A       |  true   |  false
  user_promo_codes           | primary                                       |   false    |            5 | usage_count      | N/A       |  true   |  false
(51 rows)
~~~

## See also

- [`CREATE INDEX`](create-index.html)
- [`COMMENT ON`](comment-on.html)
- [`DROP INDEX`](drop-index.html)
- [`RENAME INDEX`](rename-index.html)
- [Information Schema](information-schema.html)
- [SQL Statements](sql-statements.html)
