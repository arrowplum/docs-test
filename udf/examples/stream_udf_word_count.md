---
title: Lua UDF – Word Count
description: Lua UDF – Word Count
---

### Overview

This example walks through a variant of the algorithm published in Google's MapReduce paper, which counts the number of times each word appears in a document.

## Data Model

Each record will represent a page of a book, and will be composed of the following bins: 

* book_id – the unique identifier for the book in which this page belongs
* page_id – the unique identifier for the page.
* page_no – the page number 
* body – the body of the page.

We will assume the following secondary indexes exist:

* A NUMERIC secondary index on the book_id bin – This will allow us to query on the book identifier, so we can get the count of words for a book.
* A NUMERIC secondary index on the page_id bin – This will allow us to query on the page identifier, so we can get the count of words for a specific page.

### Code

We will create a Stream UDF that will:

* Use the `aggregate()` operation to map words to counts for each page.
* Use the `reduce()` operation to combine the maps generated from `aggregate()` into a single map.

The following is a Stream UDF named `word_count`

```
function word_count(s)
  return s : aggregate(map(), page_words) : reduce(sum_words)
end
```

The `word_count()` function applies the `aggregate()` and `reduce()` operation to stream `s`. 

First, we will utilize the `aggregate()` operation to build a mapping of words to number of instances of the word in a single page.

In this program, we will use an empty [map](/docs/udf/api/map.html) as the initial value and `page_word()` as the function for the `aggregate()` operation:

```
local function page_words(words, rec)
  -- read the 'body' bin
  local body = rec['body']
 
  -- iterate over each word in contents
  for word in string.gmatch(body, "%a+") do
     
    -- if words[word] is not nil, then use words[word], otherwise use 0 (zero)
    local count = words[word] or 0
 
    -- add 1 to the previous count
    words[word] = count + 1
  end
 
  return words
end
```

You will notice that words is used as both the first argument and return value for `page_words()`. We are essentially collecting a mapping of words to counts, which we will call words map. The second argument is a record from the database. We know it is a record because it is the first operation on the stream, which is a stream of records.

The result of the `aggregate()` operation will be a words map. If you have multiple nodes in a cluster, then you will have multiple words maps. In most instances, each node may have multiple words maps, because aggregations will be performed on partitions of data. 

Next, we will use the `reduce()` operation to merge multiple words maps into single words map.

In the is program, we will use `sum_words()` as the function for the `reduce()` operation:

```
local function sum_words(word1, words2)
  -- use map.merge() to merge two maps
  -- if the same name exists in both maps, then use the function to merge the values.
  return map.merge(word1, words2, math.sum)
end
```

We combine the two maps using `map.merge()`, which takes two maps and function. The function is used to resolve conflicts between two (name,value) pair in a map. In this example, we use `math.sum()`, which will add the two values.
