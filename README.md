# DBF
[![Build Status](https://secure.travis-ci.org/infused/dbf.png)](http://travis-ci.org/infused/dbf)
[![Gem Version](https://badge.fury.io/rb/dbf.png)](http://badge.fury.io/rb/dbf)
[![Code Climate](https://codeclimate.com/github/infused/dbf.png)](https://codeclimate.com/github/infused/dbf)
[![Dependency Status](https://gemnasium.com/infused/dbf.png)](https://gemnasium.com/infused/dbf)

DBF is a small fast library for reading dBase, xBase, Clipper and FoxPro
database files

* Project page: <http://github.com/infused/dbf>
* API Documentation: <http://rubydoc.info/github/infused/dbf/frames>
* Report bugs: <http://github.com/infused/dbf/issues>
* Questions: Email <mailto:keithm@infused.org> and put DBF somewhere in the
  subject line

## Compatibility

DBF is tested to work with the following versions of ruby:

* MRI Ruby 1.8.6, 1.8.7, 1.9.1, 1.9.2, 1.9.3, 2.0.0
* JRuby 1.6.x, 1.7.x
* REE 1.8.7
* Rubinius 2.1+

## Installation

```
gem install dbf
```

## Basic Usage

Open a DBF file:

```ruby
require 'dbf'
widgets = DBF::Table.new("widgets.dbf")
```

Enumerate all records

```ruby
widgets.each do |record|
  puts record.name
  puts record.email
end
```

Find a single record

```ruby
widget = widgets.find(6)
```

Note that find() will return nil if the requested record has been deleted
and not yet pruned from the database.

The value for a attribute can be accessed via element reference in one of three
ways

```ruby
widget["SlotNumber"]   # original field name in dbf file
widget['slot_number']  # underscored field name string
widget[:slot_number]   # underscored field name symbol
```

Get a hash of all attributes. The keys are the original column names.

```ruby
widget.attributes
# => {"Name" => "Thing1", "SlotNumber" => 1}
```

Search for records using a simple hash format. Multiple search criteria are
ANDed. Use the block form if the resulting recordset could be large, otherwise
all records will be loaded into memory.

```ruby
# find all records with slot_number equal to s42
widgets.find(:all, :slot_number => 's42') do |widget|
  # the record will be nil if deleted, but not yet pruned from the database
  if widget
    puts widget.serial_number
  end
end

# find the first record with slot_number equal to s42
widgets.find :first, :slot_number => 's42'

# find record number 10
widgets.find(10)
```

## Encodings (Code Pages)

dBase supports encoding non-english characters in different formats.
Unfortunately, the format used is not always set, so you may have to specify it
manually. For example, you have a DBF file from Russia and you are getting bad
data. Try using the 'Russion OEM' encoding:

```ruby
table = DBF::Table.new('dbf/books.dbf', nil, 'cp866')
```

See
[doc/supported_encodings.csv](docs/supported_encodings.csv)
for a full list of supported encodings.

## Migrating to ActiveRecord

An example of migrating a DBF book table to ActiveRecord using a migration:

```ruby
require 'dbf'

class Book < ActiveRecord::Base; end

class CreateBooks < ActiveRecord::Migration
  def self.up
    table = DBF::Table.new('db/dbf/books.dbf')
    eval(table.schema)

    Book.reset_column_information
    table.each do |record|
      Book.create(:title => record.title, :author => record.author)
    end
  end

  def self.down
    drop_table :books
  end
end
```

## Command-line utility

A small command-line utility called dbf is installed along with the gem.

    $ dbf -h
    usage: dbf [-h|-s|-a] filename
      -h = print this message
      -s = print summary information
      -a = create an ActiveRecord::Schema
      -c = create a csv file

Create an executable ActiveRecord schema:

    dbf -a books.dbf > books_schema.rb

Dump all records to a CSV file:

    dbf -c books.dbf > books.csv

## dBase version compatibility

The basic dBase data types are generally supported well. Support for the
advanced data types in dbase V and FoxPro are still experimental or not
supported. If you have insight into how any of the unsupported data types are
implemented, please give me a shout. FoxBase/dBase II files are not supported
at this time.

See
[doc/supported_types.markdown](docs/supported_types.markdown)
for a full list of supported column types.

## Limitations

* DBF is read-only
* Index files are not utilized

## License

Copyright (c) 2006-2013 Keith Morrison <<keithm@infused.org>>

Permission is hereby granted, free of charge, to any person
obtaining a copy of this software and associated documentation
files (the "Software"), to deal in the Software without
restriction, including without limitation the rights to use,
copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the
Software is furnished to do so, subject to the following
conditions:

The above copyright notice and this permission notice shall be
included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND,
EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES
OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND
NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT
HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY,
WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING
FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR
OTHER DEALINGS IN THE SOFTWARE.
