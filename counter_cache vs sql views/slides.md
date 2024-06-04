---
# try also 'default' to start simple
theme: apple-basic
# random image from a curated Unsplash collection by Anthony
# like them? see https://unsplash.com/collections/94734566/slidev
#background: https://cover.sli.dev
# some information about your slides, markdown enabled
title: Counter Cache vs SQL Views
info: |
  ## Counter cache bottlenecks? Unleash SQL Views!
# https://sli.dev/custom/highlighters.html
highlighter: shiki
# https://sli.dev/guide/drawing
drawings:
  persist: false
# slide transition: https://sli.dev/guide/animations#slide-transitions
transition: slide-left
# enable MDC Syntax: https://sli.dev/guide/syntax#mdc-syntax
mdc: true

layout: intro
---

# Counter cache bottlenecks? Unleash SQL Views! 

<div class="absolute bottom-10">
  <span class="font-700">
    Dorian Becker - Paris.rb June 2024
  </span>
</div>

---
transition: fade-out
layout: two-cols-header
---

# Counting sub-items
::left::

<div class="bg-gray-100 p-4  mr-6 rounded-lg">
<div class="grid grid-cols-1 gap-1">
  <div class="p-4 bg-gray-200 rounded-lg">
    <span class="text-l font-bold">Exam 1</span>
    <span class="text-sm font-bold float-right">Students: 10</span>
  </div>
  <div class="p-4 bg-gray-200 rounded-lg">
    <span class="text-l font-bold">Exam 2</span>
    <span class="text-sm font-bold float-right">Students: 10</span>
  </div>
  <div class="p-4 bg-gray-200 rounded-lg">
    <span class="text-l font-bold">Exam 3</span>
    <span class="text-sm font-bold float-right">Students: 10</span>
  </div>
 <div class="p-4 bg-gray-200 rounded-lg">
    <span class="text-l font-bold">...</span>
  </div>
</div>
</div>

::right::

````md magic-move
```ruby
class Exam
  has_many :students
end

class Student
  belongs_to :exam
end
```

```ruby
class Exam
  # students_count  :integer
  
  has_many :students, counter_cache: true
end

class Student
  belongs_to :exam
end
```
````

::v-click{at=1}
👉 Counter cache
::

---
layout: two-cols-header
---

# What if you need complex counters?


::left::
<div class="bg-gray-100 p-4  mr-6 rounded-lg">
<div class="grid grid-cols-1 gap-1">
  <div class="p-4 bg-gray-200 rounded-lg">
    <span class="text-l font-bold">Exam 1</span>
    <span class="text-sm font-bold float-right">Completed: 3/10</span>
  </div>
  <div class="p-4 bg-gray-200 rounded-lg">
    <span class="text-l font-bold">Exam 2</span>
    <span class="text-sm font-bold float-right">Completed: 9/10</span>
  </div>
  <div class="p-4 bg-gray-200 rounded-lg">
    <span class="text-l font-bold">Exam 3</span>
    <span class="text-sm font-bold float-right">Completed: 1/10</span>
  </div>
 <div class="p-4 bg-gray-200 rounded-lg">
    <span class="text-l font-bold">...</span>
  </div>
</div>
</div>  

::right::

````md magic-move

```ruby
class Exam
  has_many :students
  end

class Student
  belongs_to :exam
end
```

```ruby {9|4,9}
class Exam
  has_many :students
  
  # number_of_student_who_have_completed_the_exam ?
end

class Student
  belongs_to :exam
  enum status: { not_started: 0, started: 1, completed: 2}
end
```

```ruby {2-4,11-12|all}
class Exam
  # students_not_started_count  :integer
  # students_started_count      :integer
  # students_completed_count    :integer  

  has_many :students
end

class Student
  belongs_to :exam
  counter_culture :exam,
                  column_name: -> (model) {"students_#{model.status}_count"}
  
  enum status: { not_started: 0, started: 1, completed: 2}
end
```
````

::v-click{at=3}
 `counter_culture` gem
::

---
layout: center
class: text-center
---

# Dorian Becker
CTO @ Evalmee

---
layout: two-cols-header
---

# Let's add new features!


::left::

<div class="bg-gray-100 p-4  mr-6 rounded-lg">
<div class="grid grid-cols-1 gap-1">
  <div class="p-4 bg-gray-200 rounded-lg">
    <span class="text-l font-bold">Exam 1</span>
    <span class="text-sm font-bold float-right" v-click="2">Open sessions: 3/10</span>
  </div>
  <div class="p-4 bg-gray-200 rounded-lg">
    <span class="text-l font-bold">Exam 2</span>
    <span class="text-sm font-bold float-right" v-click="2">Open sessions: 9/10</span>
  </div>
  <div class="p-4 bg-gray-200 rounded-lg">
    <span class="text-l font-bold">Exam 3</span>
    <span class="text-sm font-bold float-right" v-click="2">Open sessions: 1/10</span>
  </div>
 <div class="p-4 bg-gray-200 rounded-lg">
    <span class="text-l font-bold">...</span>
  </div>
</div>
</div> 

::right::
````md magic-move
```ruby
class Exam
  has_many :sessions
end

class Session
  #  start_at  :datetime
  #  end_at    :datetime
  
  belongs_to :exam
end
```
```ruby {11-12}
class Exam
  has_many :sessions
end

class Session
  #  start_at  :datetime
  #  end_at    :datetime
  
  belongs_to :exam

  scope :open, 
        -> { where("start_at <= ? AND end_at >= ?", Time.zone.now, Time.zone.now) }
end
```

```ruby {3,11-13}
class Exam
  has_many :sessions
  # open_sessions_count ?
end

class Session
  #  start_at  :datetime
  #  end_at    :datetime
  
  belongs_to :exam

  scope :open, 
        -> { where("start_at <= ? AND end_at >= ?", Time.zone.now, Time.zone.now) }
end
```
````

---
layout: statement
---

# How to count items depending on a date?

---

# SQL Views
## What is it?

A SQL View is a virtual table based on the result of an SQL statement.

```sql
CREATE VIEW view_name AS
SELECT column1, column2, ...
FROM table_name
WHERE condition;
```

---

# SQL Views
## Real example

````md magic-move {lines: true}
```sql
SELECT exams.id as id,
   (
      SELECT COUNT(*)
      FROM students
      WHERE students.exam_id = exams.id
   ) as students_count
FROM exams

```

```sql
SELECT exams.id as id,
   (
      SELECT COUNT(CASE WHEN start_at <= NOW() AND (end_at >= NOW() OR end_at IS NULL) THEN 1 END) AS open_count,
      FROM sessions
      WHERE sessions.exam_id = exams.id
   ) as open_sessions_count
FROM exams

```


```sql {1-5|6-12|13-19}
students_cte AS (
  SELECT students.exam_id, COUNT(*) AS count
  FROM students
  GROUP BY students.exam_id
),
sessions_cte AS (
  SELECT
    exam_id,
    COUNT(CASE WHEN start_at <= NOW() AND (end_at >= NOW() OR end_at IS NULL) THEN 1 END) AS open_count,
  FROM sessions
  GROUP BY exam_id
)
SELECT
  exams.id as id,
  coalesce(scores_cte.count, 0) as scores_count,
  coalesce(sessions_cte.open_count, 0) as open_sessions_count,
FROM exams
LEFT JOIN students_cte ON students_cte.exam_id = exams.id
LEFT JOIN sessions_cte ON sessions_cte.exam_id = exams.id

```

```sql {9,11,12,19,21,22}
students_cte AS (
  SELECT students.exam_id, COUNT(*) AS count
  FROM students
  GROUP BY students.exam_id
),
sessions_cte AS (
  SELECT
    exam_id,
    COUNT(*) AS total_count,
    COUNT(CASE WHEN start_at <= NOW() AND (end_at >= NOW() OR end_at IS NULL) THEN 1 END) AS open_count,
    COUNT(CASE WHEN start_at > NOW() OR (end_at < NOW() AND end_at IS NOT NULL) THEN 1 END) AS closed_count,
    COUNT(CASE WHEN end_at < NOW() AND end_at IS NOT NULL THEN 1 END) AS ended_count
  FROM sessions
  GROUP BY exam_id
)
SELECT
  exams.id as id,
  coalesce(scores_cte.count, 0) as scores_count,
  coalesce(sessions_cte.total_count, 0) as sessions_count,
  coalesce(sessions_cte.open_count, 0) as open_sessions_count,
  coalesce(sessions_cte.closed_count, 0) as closed_sessions_count,
  coalesce(sessions_cte.ended_count, 0) as ended_sessions_count
FROM exams
LEFT JOIN students_cte ON students_cte.exam_id = exams.id
LEFT JOIN sessions_cte ON sessions_cte.exam_id = exams.id

```
````

---

# SQL Views
## How to create one in Rails?

2 Solutions:
::v-clicks
- Move from `schema.rb` to `structure.sql` 
  - write the SQL View in migration files
  - create an associated model
- Use <span v-mark="{ at: 3, color: 'red', type: 'circle' }">
    <code>scenic</code> gem
  </span>
  - files dedicated to SQL Views migration
  - associated model created automatically
::

---

# Scenic gem

```bash {1|7|6|3}
rails g scenic:model exam_stats
   invoke  active_record
   create    app/models/exam_stats.rb
   invoke    rspec
    create      spec/models/exam_stats_spec.rbs
   create  db/views/exam_stats_v01.sql
   create  db/migrate/[TIMESTAMP]_create_exam_stats_view.rb
```

<div v-click="1" >
```ruby
# db/migrate/[TIMESTAMP]_create_exam_stats_view.rb
class CreateExamStatsView < ActiveRecord::Migration[7.1]
  def change
    create_view :exam_stats
  end
end
```
</div>


<div v-click="2" >
```sql
-- db/views/exam_stats_v01.sql
students_cte AS (
  SELECT students.exam_id, COUNT(*) AS count
  FROM students
  GROUP BY students.exam_id
),
-- ...
```
</div>

---

# Model associated with the SQL View

```ruby
# app/models/exam_stats.rb

# == Schema Information
#
# Table name: exam_stats
#
#  id                    :integer          primary key
#  closed_sessions_count :bigint
#  ended_sessions_count  :bigint
#  open_sessions_count   :bigint
#  sessions_count        :bigint
#  students_count        :bigint

class ExamStats < ApplicationRecord

  self.primary_key = :id

  belongs_to :exam, inverse_of: :stat, foreign_key: :id

  def readonly? = true
end
```

---
layout: two-cols-header
---
# SQL View in Rails

::left::
# 👍 Pros
- handle **complex** counters
- **no cache** to update
- no **concurrency** issue

::right::
# 👎 Cons

- logic moved from your model to the database
- a bit slower than a counter cache

---
layout: center
class: text-center
---

# Thank you!

[@blaked_84](https://x.com/blaked_84) · [evalmee.com](https://evalmee.com) · [beckr.fr](https://beckr.fr)

===

[counter_culture gem](https://github.com/magnusvk/counter_culture)  · 
[scenic gem](https://github.com/scenic-views/scenic)
