# Data Engineering Coding Challenges

## Judgment Criteria

- Beauty of the code (beauty lies in the eyes of the beholder)
- Testing strategies
- Basic Engineering principles

## Problem 1

### Parse fixed width file

- Generate a fixed width file using the provided spec (offset provided in the spec file represent the length of each field).
- Implement a parser that can parse the fixed width file and generate a delimited file, like CSV for example.
- DO NOT use python libraries like pandas for parsing. You can use the standard library to write out a csv file (If you feel like)
- Language choices (Python or Scala)
- Deliver source via github or bitbucket
- Bonus points if you deliver a docker container (Dockerfile) that can be used to run the code (too lazy to install stuff that you might use)
- Pay attention to encoding

"""
## Sample spec file (json file)
{
    "ColumnNames":"f1, f2, f3, f4, f5, f6, f7, f8, f9, f10",
    "Offsets":"3,12,3,2,13,1,10,13,3,13",
    "InputEncoding":"windows-1252",
    "IncludeHeader":"True",
    "OutputEncoding":"utf-8"
}
"""

import io
import re
import argparse
import json
import csv
import logging
import unittest


def split_line_by_fixed_widths(textline = '', offsets = []):
    line = textline
    delimiter = ';'

    for idx, n in enumerate(offsets):

        if idx == len(offsets) - 1:
            continue

        line = re.sub(r"^(.{%d})()" % (n + idx), r"\1%s" % delimiter, line)
        logging.debug(line)

    line = [n.strip() if len(n.strip()) > 0 else '_' for n in line.split(delimiter)]

    return line


def parse_spec(spec):
    column_names = []
    offsets = []
    include_header = False
    input_encoding = None
    output_encoding = None

    try:
        with open(spec) as spec_file:
            spec = json.load(spec_file)
            column_names = [s.encode("utf-8") for s in re.split(r"\s+", spec.get('ColumnNames').replace(r",", ""))]
            offsets = [int(s) for s in re.split(r"\D+", spec.get('Offsets'))]
            input_encoding = spec.get('InputEncoding')
            include_header = spec.get('IncludeHeader')
            output_encoding = spec.get('OutputEncoding')

        return (
            column_names,
            offsets,
            include_header,
            input_encoding,
            output_encoding,
        )

    except Exception as err:
        logging.error('Cannot parse the spec')
        logging.error(err)


def run(spec, inputfile):

    column_names, offsets, include_header, input_encoding, output_encoding = parse_spec(spec)

    # We calculate the offsets to the line-beginning
    reduced_offsets = []
    for idx, width in enumerate(offsets):
        distance = width if idx == 0 else width + reduced_offsets[idx - 1]
        reduced_offsets.append(distance)

    try:
        with open('result.csv', 'w') as csv_file:
            writer = csv.writer(csv_file, delimiter=';')

            with open(inputfile, 'r') as f:

                if include_header:
                    writer.writerow(column_names)

                for line_index, line in enumerate(f.readlines()):

                    if line_index == 1 or len(line) == 0:
                        continue

                    splitted_line = split_line_by_fixed_widths(line, reduced_offsets)
                    writer.writerow(splitted_line)

                f.close()
            csv_file.close()
    except Exception as err:
        logging.error('File IO error')
        logging.error(err)



if __name__ == '__main__':
    parser = argparse.ArgumentParser(description='Text to csv')
    parser.add_argument('spec', metavar='F', type=str, help='spec')
    parser.add_argument('file', metavar='F', type=str, help='textfile')
    args = parser.parse_args()
    spec = args.spec
    file = args.file
    run(spec, file)

# parser_test.py
import unittest
from parser import *


class MyTest(unittest.TestCase):

    def test_split_line(self):
        line = "abcd efgh 1123"
        offsets = [4, 9, 14]
        expected = ['abcd', 'efgh', '1123']
        self.assertEqual(split_line_by_fixed_widths(line, offsets), expected)

    def test_split_line_with_empty_value(self):
        line = "abcd efgh 1123   "
        offsets = [4, 9, 14, 17]
        expected = ['abcd', 'efgh', '1123', '_']
        self.assertEqual(split_line_by_fixed_widths(line, offsets), expected)

        line = "abcd efgh    1123"
        offsets = [4, 9, 12, 17]
        expected = ['abcd', 'efgh', '_', '1123']
        self.assertEqual(split_line_by_fixed_widths(line, offsets), expected)


if __name__ == '__main__':
    unittest.main()

## Problem 2

### Data processing

- Generate a csv file containing first_name, last_name, address, date_of_birth
- Process the csv file to anonymise the data
- Columns to anonymise are first_name, last_name and address
- You might be thinking  that is silly
- Now make this work on 2GB csv file (should be doable on a laptop)
- Demonstrate that the same can work on bigger dataset
- Hint - You would need some distributed computing platform

## Choices

- Any language, any platform
- One of the above problems or both, if you feel like it.
