<!DOCTYPE html>
<html>
<head>
<meta name="viewport" content="width=device-width; initial-scale=1">
<title>Rewriting the SAT vocab tester.</title>
<link rel="stylesheet" type="text/css" href="http://maxcdn.bootstrapcdn.com/bootstrap/3.2.0/css/bootstrap.min.css">

<style type="text/css">
body{font-family: Source Sans Pro, Sans, Helvetica Neue, sans-serif;}
</style>
</head>
<body> 
<div class="container">

# satv2

The [old code](https://github.com/pharpend/sat-vocab-test) is so terrible that I
think it's time to write a replacement. What better way to show you how it works
than to write it from scratch?

I'll be following this tutorial pretty much to the letter. You can see what I'm
doing [on GitHub](https://github.com/pharpend/satv2).

# First thing, get stuff.

I'm going to write this in Python 3, simply because I have my editor set up to
use Python 3 for completion and what not. Also, Python 3 has been around for,
what, 8 years? Time to upgrade.

So, install Python 3, and git. Because git is nice.

You'll also want to make a directory for the new program.

    mkdir satv2
    cd satv2
    git init

# Get the data

The ~~first~~ second thing to do is to look at the data - the data is stored in
a
[poorly formatted text file](https://raw.githubusercontent.com/pharpend/sat-vocab-test/master/sat_vocab_text.txt).

We need to convert that text into a more Python-friendly format. JSON is my
vote. Python also has a good XML library, but JSON is more familiar to me.

To quickly download the data file, run

    curl https://raw.githubusercontent.com/pharpend/sat-vocab-test/master/sat_vocab_text.txt -o definitions.txt

Next, we're going to commit the definitions file

    git add definitions.txt
    git commit -m "Initial commit, contains definitions file."

# Convert the data to JSON

We're going to write a little script to convert that text to JSON. The first
thing to do is to read the file.

```python
def_handle = open("definitions.txt", "r")
def_text = def_handle.read()
def_handle.close()
```
We're going to want to put all of the lines into a list.

```python
def_lines = def_text.splitlines()
```

Here's the output of `def_lines[:5]`:

    ['abase v. To lower in position, estimation, or the like; degrade.',
     'abbess n. The lady superior of a nunnery. ',
     'abbey n. The group of buildings which collectively form the dwelling-place of a society of monks or nuns',
     'abbot n. The superior of a community of monks. ',
     'abdicate v. To give up (royal power or the like). ']


Next, we're going to want to split each line into individual words.

```python
def_words = [line.split() for line in def_lines]
```
Here's the first 5 elements:

    [['abase', 'v.', 'To', 'lower', 'in', 'position,', 'estimation,', 'or', 'the', 'like;', 'degrade.'],
     ['abbess', 'n.', 'The', 'lady', 'superior', 'of', 'a', 'nunnery.'],
     ['abbey', 'n.', 'The', 'group', 'of', 'buildings', 'which', 'collectively', 'form', 'the', 'dwelling-place', 'of', 'a', 'society', 'of', 'monks', 'or', 'nuns'],
     ['abbot', 'n.', 'The', 'superior', 'of', 'a', 'community', 'of', 'monks.'],
     ['abdicate', 'v.', 'To', 'give', 'up', '(royal', 'power', 'or', 'the', 'like).']]

Next, we're going to want to group those words. The first word is the
`word`. The second word is the `pos`, or "part of speech". The remainder is the
`def`, or "definition."

    def_groups = [(g[0], g[1][:-1], " ".join(g[2:])) for g in def_words]

Here are the first 5 entries

    [('abase', 'v', 'To lower in position, estimation, or the like; degrade.'),
     ('abbess', 'n', 'The lady superior of a nunnery.'),
     ('abbey', 'n', 'The group of buildings which collectively form the dwelling-place of a society of monks or nuns'),
     ('abbot', 'n', 'The superior of a community of monks.'),
     ('abdicate', 'v', 'To give up (royal power or the like).')]

Next, we want to turn each of these entries into an associative list, or
"dictionary" in Python terms.

    keys = ("word", "pos", "def")
    def_dicts = [dict(zip(keys, vals)) for vals in def_groups]

Here are the first 5 entries

    [{'pos': 'v.',
      'def': 'To lower in position, estimation, or the like; degrade.',
      'word': 'abase'},
     {'pos': 'n.', 'def': 'The lady superior of a nunnery.', 'word': 'abbess'},
     {'pos': 'n.',
      'def': 'The group of buildings which collectively form the dwelling-place of a society of monks or nuns',
      'word': 'abbey'},
     {'pos': 'n.',
      'def': 'The superior of a community of monks.',
      'word': 'abbot'},
     {'pos': 'v.',
      'def': 'To give up (royal power or the like).',
      'word': 'abdicate'}]

We're going to create two separate JSON files - one that is prettily encoded, so
humans can read it, and one that is compressed, with no line breaks or
indentation, so our program can read it quickly.

    import json
    pretty_encoder = json.JSONEncoder(indent=2)
    ugly_encoder = json.JSONEncoder(separators=(",", ":"))

To encode

    pretty_encoded = pretty_encoder.encode(def_dicts)
    ugly_encoded = ugly_encoder.encode(def_dicts)

Here are the first 256 characters of `pretty_encoded`.

    [
      {
        "pos": "v",
        "def": "To lower in position, estimation, or the like; degrade.",
        "word": "abase"
      },
      {
        "pos": "n",
        "def": "The lady superior of a nunnery.",
        "word": "abbess"
      },
      {
        "pos": "n",
        "def": "The group of bu

By contrast, here are the first 256 characters of `ugly_encoded`.

    [{"pos":"v","def":"To lower in position, estimation, or the like; degrade.","word":"abase"},{"pos":"n","def":"The lady superior of a nunnery.","word":"abbess"},{"pos":"n","def":"The group of buildings which collectively form the dwelling-place of a society

Next, we're going to want to write this JSON to files.

    pretty_handle = open("defs-pretty.json", "w")
    pretty_handle.write(pretty_encoded)
    pretty_handle.close()

    ugly_handle = open("defs-ugly.json", "w")
    ugly_handle.write(ugly_encoded)
    ugly_handle.close()

Here's the entire script, in a file called `json-migrator.py`.

    def_handle = open("definitions.txt", "r")
    def_text = def_handle.read()
    def_handle.close()

    def_lines = def_text.splitlines()
    def_words = [line.split() for line in def_lines]

    def_groups = [(g[0], g[1][:-1], " ".join(g[2:])) for g in def_words]

    keys = ("word", "pos", "def")
    def_dicts = [dict(zip(keys, vals)) for vals in def_groups]

    import json
    pretty_encoder = json.JSONEncoder(indent=2)
    ugly_encoder = json.JSONEncoder(separators=(",", ":"))

    pretty_encoded = pretty_encoder.encode(def_dicts)
    ugly_encoded = ugly_encoder.encode(def_dicts)

    pretty_handle = open("defs-pretty.json", "w")
    pretty_handle.write(pretty_encoded)
    pretty_handle.close()

    ugly_handle = open("defs-ugly.json", "w")
    ugly_handle.write(ugly_encoded)
    ugly_handle.close()

## Commit your changes

Next, we're going to commit everything. We no longer need the old text file, so
we'll go ahead and delete it.

    git add .
    git rm definitions.txt
    git commit -m "Converted the definitions file to JSON."

We no longer need the converter script, so we'll delete it.

    git rm json-migrator.py
    git commit -m  "Removed the migrator script."

# Writing the actual tester

This section concerns writing the thing that actually quizzes people.

First, we're going to open a file called `satv2.py`. Since this is production
code, we do need to document it. We'll first add in a little prelude

    #!/usr/bin/env python3

    """satv2 - version 2 of the SAT tester.

    This program quizzes people on SAT vocabulary.
    """

    import json

## Marshal the JSON data into a Python data type

First, we should make a dummy `main` function.

    def main():
        return

    if '__main__' == __name__:
        main()

From now on, each successive code snippet listed here should precede the others
in your file.

This program doesn't actually do anything so far. The first thing we'll want to
do is write a function to marshal the file text into our `Vocab` type.

    def marshal_json_text(text):
        """Marshal json text into dict"""
        defs_dicts = json.loads(text)
        return defs_dicts

We'll also want a function to read the `defs-ugly.json` file.

    def read_ugly_file():
        """Read the defs-ugly.json file."""
        ugly_handle = open("defs-ugly.json", "r")
        ugly_text = ugly_handle.read()
        ugly_handle.close()
        return ugly_text

We'll want a function that composes the two.

    def marshal_ugly_file():
        """Read the defs-ugly.json file, marshal it into a bunch dicts."""
        return marshal_json_text(read_ugly_file())

## In glorious Haskell

I'm a Haskell programmer at heart, so I'm going to digress, for my own sake. In
Haskell, I would write something like

    data PartOfSpeech = Noun | Verb | Adjective | Adverb | etc.
    data Vocab = Vocab { word         :: Word
                       , partOfSpeech :: Pos
                       , definition   :: Definition
                       }
    type Word       = Text
    type Pos        = PartOfSpeech
    type Definition = Text
    type WordMap    = Map Word Vocab
    type DefMap     = Map Def Vocab

and then write `FromJSON` instances for `Word`, `Pos`, and `Definition`. Why
can't Python be Haskell?

## Back to Python

Anyway, the equivalent in Python would, I guess, be a dictionary mapping
either 

* The `word` to the `vocab`
* The `def` to the `vocab`

So, here are the two functions:

    # So, here's the function to make the word map.
    def mk_word_map(vocabs):
        """Takes a bunch of `Vocab` objects, marshals them into the Python equivalent
        of `WordMap`."""
        wordmap = { v.word: v
                    for v in vocabs }
        return wordmap

    # We'll also want a function that makes a definition map
    def mk_def_map(vocabs):
        """Takes a bunch of `Vocab` objects, marshals them into the Python equivalent
        of a `DefMap`."""
        defmap = { v.definition: v
                   for v in vocabs }
        return defmap

### Librarizing

The next thing to do is to move the JSON stuff to it's own file.  Here are the
various files. First, `satjson.py`

    """JSON interface for satv2"""

    import json

    def marshal_json_text(text):
        """Marshal json text into a dict"""
        return json.loads(text)

    def read_ugly_file():
        """Read the defs-ugly.json file."""
        ugly_handle = open("defs-ugly.json", "r")
        ugly_text = ugly_handle.read()
        ugly_handle.close()
        return ugly_text

    def marshal_ugly_file():
        """Read the defs-ugly.json file, marshal it into a bunch of dicts."""
        return marshal_json_text(read_ugly_file())

Next, `satv2.py`

    #!/usr/bin/env python3

    """satv2 - version 2 of the SAT tester.

    This program quizzes people on SAT vocabulary.
    """

    def main():
        """The main function, runs everything"""
        return

    if '__main__' == __name__:
        main()

## A little bit of testing

We're going to modify `main` to make sure we can read the file

    def main():
        """The main function, runs everything"""
        vs = satjson.marshal_ugly_file()
        pprint.pprint(vs[:5])

You'll also want to add `import satjson, pprint` to the top of the file
    
Run `python satv2.py`, and the output should look a little bit like this:

    [{'def': 'To lower in position, estimation, or the like; degrade.',
      'pos': 'v',
      'word': 'abase'},
     {'def': 'The lady superior of a nunnery.', 'pos': 'n', 'word': 'abbess'},
     {'def': 'The group of buildings which collectively form the dwelling-place '
             'of a society of monks or nuns',
      'pos': 'n',
      'word': 'abbey'},
     {'def': 'The superior of a community of monks.', 'pos': 'n', 'word': 'abbot'},
     {'def': 'To give up (royal power or the like).',
      'pos': 'v',
      'word': 'abdicate'}]

You'll want to commit your changes

    git add .
    git commit -m "Marshal JSON back into Python types."

# What next?

Next, you'll want to add an interface, that actually quizzes people. So far,
I've shown you how to get the data from files. It should be pretty obvious how
to proceed from here.

How you design your interface, or what type of interface, is entirely up to you.

You can see the work as of now [on GitHub](https://github.com/pharpend/satv2).

Once you design that, you need to pick a license, write a README, and write a
lot of documentation on usage.

</div>
<script src="http://ajax.googleapis.com/ajax/libs/jquery/1.11.1/jquery.min.js"></script>
<script src="http://maxcdn.bootstrapcdn.com/bootstrap/3.2.0/js/bootstrap.min.js"> </script>
</body>
</html>
