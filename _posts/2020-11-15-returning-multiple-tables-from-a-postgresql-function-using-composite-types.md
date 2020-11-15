---
layout: post
title: "Returning Multiple Tables from a PostgreSQL Function using Composite Types"
date: 2020-11-15 14:00:00 +0000
tags: [database,postgres]
---
[PostgreSQL functions](https://www.postgresql.org/docs/13/sql-createfunction.html) provide a convenient way to perform a set of operations with only one call to the database.
Data from multiple tables can be returned by a function, however, complexity is introduced when trying to use this data in an application due to its format.
This post explores how to return data from multiple tables and convert the data to a more useful format for an application.

## Defining the function

To start, a function is created that finds and returns the students for a particular class, as shown below:

```sql
CREATE FUNCTION get_class(class integer) RETURNS SETOF students AS $$
    BEGIN
        RETURN QUERY SELECT *
        FROM students
        WHERE students.class_id = class;
    END;
$$ LANGUAGE plpgsql;
```

This function, called `get_class`, has one argument `class` which represents the identifier for the class.
The return data type of the function is set to the composite type of the `students` table with the `SETOF` modifier, stating that a set of items is returned.

> "Whenever you create a table, a composite type is automatically created, with the same name as the table, to represent the table's row type." - [postgresql](https://www.postgresql.org/docs/9.6/rowtypes.html)

This function returns the following:

```csv
id,name,joined_at,class_id
1,David,2020-11-15 11:02:33.450739+00,1
2,John,2020-11-15 11:02:33.450739+00,1
3,Hannah,2020-11-15 11:02:33.450739+00,1
```

## Returning multiple tables

This function can be expanded to included information on the teacher for a class, with the new `teachers` table having the same structure as the `students` table.
A [composite type](https://www.postgresql.org/docs/13/rowtypes.html) called `class` is created with two fields for the students and teacher data.
The `teacher` field type is set to the `teachers` composite type, while the `students` field is set to `json`, allowing multiple rows of students to be returned alongside the teacher data.

```sql
CREATE TYPE class AS (
    teacher teachers,
    students json
);
```

The `get_class()` function is changed to provide the required data for its new return type, `class`.
First, the students for the class are gathered and converted to JSON.
Second, the teacher data is gathered and assigned to the `class_data` variable.
Finally, the `class_data` variable is returned by the function.

```sql
CREATE FUNCTION get_class(class integer) RETURNS class AS $$
    DECLARE
        teacher_data teachers;
        class_data class;
    BEGIN
        SELECT array_to_json(array_agg(students_result))
        INTO class_data.students
        FROM (
            SELECT *
            FROM students
            WHERE students.class_id = class
        ) students_result;

        SELECT *
        INTO teacher_data
        FROM teachers
        WHERE teachers.class_id = class;

        class_data.teacher := teacher_data;

        RETURN class_data;
    END;
$$ LANGUAGE plpgsql;
```

The DECLARE statement includes the teacher_data variable, which is used to temporarily house the teacher data while waiting to be assigned to the `class_data` variable.

This function now returns the following:

```csv
teacher,students
(1,Matthew,"2020-11-15 11:25:51.382269+00",1),
[
    {"id":1,"name":"David","joined_at":"2020-11-15T11:02:33.450739+00:00","class_id":1},
    {"id":2,"name":"John","joined_at":"2020-11-15T11:02:33.450739+00:00","class_id":1},
    {"id":3,"name":"Hannah","joined_at":"2020-11-15T11:02:33.450739+00:00","class_id":1}
]
```

## Introducing a third table

A third table can be added, for example the `classes` table to provide information about the class.
First, the `class` composite type is changed to include the class data.

```sql
CREATE TYPE class AS (
    class json,
    teacher json,
    students json
);
```

Second, the `get_class` function is changed to also query the `classes` table:

```sql
CREATE FUNCTION get_class(class integer) RETURNS class AS $$
    DECLARE
        class_data class;
    BEGIN
        SELECT
            row_to_json(classes),
            row_to_json(teachers),
            array_to_json(array_agg(students))
        INTO class_data
        FROM classes
        INNER JOIN teachers ON teachers.class_id = classes.id
        INNER JOIN students ON students.class_id = classes.id
        WHERE classes.id = class
        GROUP BY
            classes,
            teachers;

        RETURN class_data;
    END;
$$ LANGUAGE plpgsql;
```

The query result is grouped by the `classes` and `teachers` columns as the `array_agg` [aggregration function](https://www.postgresql.org/docs/13/functions-aggregate.html) is used in the SELECT statement.
Additionally, the `row_to_json` function is used to convert the data for classes and teachers to JSON, making it easier to use in an application.

The returned data from this function is:

```csv
class,teacher,students
{"id":1,"name":"History"},
{"id":1,"name":"Matthew","joined_at":"2020-11-15T11:25:51.382269+00:00","class_id":1},
[
    {"id":1,"name":"David","joined_at":"2020-11-15T11:02:33.450739+00:00","class_id":1},
    {"id":2,"name":"John","joined_at":"2020-11-15T11:02:33.450739+00:00","class_id":1},
    {"id":3,"name":"Hannah","joined_at":"2020-11-15T11:02:33.450739+00:00","class_id":1}
]
```