# JsonGet

Small ansi C library to retrieve values from json 

__Features:__

 * No dynamic memory allocations! -  All functions work only with original json buffer
 * On-demand single pass parsing! - Parse only requested branch
 * No dependencies! -   Even from standart C libs
 * Easy to use! - only 2 files

Library doesn't try to construct full syntax tree of json. It parses only requested branch, so it can work with partly-corrupted json data

## Example 1: Hello, World!

```cpp
#include <stdio.h>
#include "jsonget.h"

int main()
{
    char str_buf[100];
    int len;
    JsonGetCursor root = jsonget("{\"Hello\": \"world\"}");
    jsonget_string(jsonget_move_key(root, "Hello"), str_buf, sizeof(str_buf), &len);
    printf("Hello, %s!", str_buf); // Prints: Hello, World!
}
```

## Usage

To start work with json, you should create cursor on json buffer

```cpp
JsonGetCursor root = jsonget("{\"Hello\": \"world\"}");
```

Cursor is a pointer to some value in json. There are 4 functions to "move" pointer:

* `JsonGetCursor jsonget_move_key(JsonGetCursor cursor, char* key)` - return cursor to value of _key_ object field;

* `JsonGetCursor jsonget_move_index(JsonGetCursor cursor, int index)` - return cursor to _index_ element of array, or to _index_ pair (key: value) of object;

* `JsonGetCursor jsonget_move_next(JsonGetCursor cursor)` - return cursor to next element in array, or to next pair in object;

* `JsonGetCursor jsonget_move_pair_value(JsonGetCursor cursor)` - return cursor to value of pair.

If you try to move cursor to non-existent field or index, special invalid cursor will be returned (with `type == JSONGET_INVALID`).

In our Hello-world example, select field "Hello": 

```cpp
cursor = jsonget_move_key(root, "Hello")
```

Now let's get value of cursor. There are 5 functions, each return 1, when success, and 0 when they fail:

* `int jsonget_int(JsonGetCursor cursor, int* out_int)` - get integer value from cursor (work with cursor types: `JSONGET_NULL` - return 0, `JSONGET_BOOLEAN` - return 1 if true otherwise 0, `JSONGET_INTEGER` - return actual value, `JSONGET_DOUBLE` - return floor(double value));

* `int jsonget_double(JsonGetCursor cursor, double* out_double)` - get float-point value from cursor (work with cursor types: `JSONGET_INTEGER` and `JSONGET_DOUBLE`);

* `int jsonget_raw(JsonGetCursor cursor, char** out_string_start, int* out_length)` - get raw representation of value. E.g. `[1, 2]` for `JSONGET_ARRAY`, `{"key": 2}` for JSONGET_OBJECT and `"string"` for `JSONGET_STRING`;

* `int jsonget_raw_copy(JsonGetCursor cursor, char* dest_buffer, int buffer_size, int *out_real_length)` - copy result of `jsonget_raw` function to _dest_buffer_;

* `int jsonget_string(JsonGetCursor cursor, char* dest_buffer, int buffer_size, int *out_real_length)` - get string value from cursor.

Get value and print result:

```cpp
jsonget_string(cursor, str_buf, sizeof(str_buf), &len);
printf("Hello, %s!", str_buf);
```

There are also other helpful functions: 

* `int jsonget_type(JsonGetCursor cursor)` - return type of cursor (or just use cursor.type field);

* `int jsonget_isnull(JsonGetCursor cursor)` - return 1 if cursor type is `JSONGET_INVALID` or `JSONGET_NULL`, otherwise return 0;

* `int jsonget_istrue(JsonGetCursor cursor)` - return 1 if cursor type is `JSONGET_BOOLEAN` and value is true, otherwise return 0

* `int jsonget_array_count(JsonGetCursor cursor)` - return number of elements in array, number of pairs in object;

* `int jsonget_string_compare(JsonGetCursor cursor, char* str2);` - compare string value of cursor with specified string

## Example 2: Iterate over json object fields or array elements

```cpp
int real_len;
char key[255];
char buffer[255];
JsonGetCursor root = jsonget("{\"k\": \"v\", \"a\": [10, 20]}"); 

JsonGetCursor cur = jsonget_move_index(root, 0);
while (cur.type != JSONGET_INVALID)
{
    jsonget_string(cur, key, sizeof(key), &real_len); 
    jsonget_string(jsonget_move_pair_value(cur), buffer, sizeof(buffer), &real_len); 
    printf("%s: %s\n", key, buffer);
    cur = jsonget_move_next(cur);
}

// Prints:
// k: v
// a: [10, 20]
```

## License

```
The MIT License (MIT)

Copyright (c) 2013 Elantcev Mikhail

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in
all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
THE SOFTWARE.
```