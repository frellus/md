#!/usr/bin/env python3

import argparse
import os
import os.path
from pathlib import Path
import re
import sys

# Constants
INDEX_HEADER_FILE     = "README-header.md"
INDEX_FOOTER_FILE     = "README-footer.md"
INDEX_OUTPUT          = "README.md"
MD_TEMPLATE           = "TEMPLATE.md"
MD_IGNORE             = ".md-ignore"
MAX_CATEGORY_DEPTH    = 5
MARKDOWN_ROOT_DEFAULT = "./"

VERSION               = "1.0 - source code: https://github.com/frellus/md.git (author: Greg Gallagher)"

# Global variables
total_md_files_indexed = 0

#
# Main Function
#
def main():

    # get the program name
    progname = sys.argv[0]

    # parse the CLI arguments
    parser = argparse.ArgumentParser()
    parser.add_argument("-i", "--index",    action='store_true',   help=f'generate an index into {INDEX_OUTPUT}')
    parser.add_argument("-n", "--new", metavar='title', dest='title',            help="create a new document with the title")
    parser.add_argument("-c", "--category",                        help="category for this document (specify new or existing, with '/' as path seperator -- ex 'python/testing'")
    parser.add_argument("-r", "--root",                            help="document root (also by ENV MARKDOWN_ROOT)")
    parser.add_argument("-v", "--version",  action='store_true',   help="display version")
    args = parser.parse_args()
    
    if args.version:
        print(f'{progname} - {VERSION}')
        sys.exit()

    # Change to the MARKDOWN_ROOT (or --root) if specified, otherwise default of '.' CWD
    if args.root:
        markdown_root = args.root
    elif os.environ.get('MARKDOWN_ROOT'):
        markdown_root = os.environ.get('MARKDOWN_ROOT')
    else:
        markdown_root = MARKDOWN_ROOT_DEFAULT

    try:
        os.chdir(markdown_root)
    except Exception as e:
        print(f'error: accessing document root: ', e)
        sys.exit(3)

    # Error out of the root has a ".md-ignore" file so we don't accidentally overwrite a hand-edited README.md file
    if os.path.exists(MD_IGNORE):
        print(f'Exiting due to {MD_IGNORE} file in markdown root directory {markdown_root} (will not overwrite {INDEX_OUTPUT})')
        sys.exit(4)

    # Check the action to perform
    if args.index:
        # generate an index file
        print(f'Generating markdown index to file: {INDEX_OUTPUT}')
        generate_index_output_file()
    elif args.title:
        # check a title was included
        if not args.title:
            print('error: you must specify a title for this document')
            parser.print_help()
            sys.exit(100)
        
        if not args.category:
            print('error: you must specify a category for this document (can include a sub-category such as \"python/testing\")')
            parser.print_help()
            sys.exit(100)

        # get the file path, filename and title string
        md_file_path = create_category_path(args.category.lower()) + '/' + title_to_filename(args.title)
        title_string = capitalize_title_words(args.title)
        
        # Generate the Markdown file 
        if markdown_root == './':
            print(f'Generating new Markdown file with title \"{title_string}\" --> {md_file_path}')
        else:
            print(f'Generating new Markdown file with title \"{title_string}\" --> {markdown_root}/{md_file_path}')

        if not generate_new_md_file(md_file_path, title_string, MD_TEMPLATE):
            sys.exit(10)
        else:
            # generate a new INDEX file
            generate_index_output_file()

    else:
        parser.print_help()
        
    sys.exit(0)


def generate_new_md_file(md_file_path, title_string, MD_TEMPLATE):

    # make sure the file doesn't already exist
    if os.path.isfile(md_file_path):
        print(f'error: markdown file already exists, not overwriting!')
        return False

    # check the depth of the file is not more than MAX_CATEGORY_DEPTH levels deep
    file_path_depth = len(md_file_path.split("/"))
    if file_path_depth > MAX_CATEGORY_DEPTH:
        print(f'error: catagory path is too deep, must be < {MAX_CATEGORY_DEPTH} levels')
        return False

    try:
        # open the new markdown file and add the title for the time
        md_file = open(md_file_path, 'w')
        md_file.write(f'# {title_string}\n\n')
        # import the 
        template_lines = readfile(MD_TEMPLATE)
        for l in list(template_lines):
            l.rstrip()
            md_file.writelines(l)
        return True
    except Exception as e:
        print(f'error: couldn\'t open file for writing: ', e)
        return False


    
#
# Actions
#
def generate_index_output_file():
    
    header_lines = readfile(INDEX_HEADER_FILE)
    footer_lines = readfile(INDEX_FOOTER_FILE)
    
    # open the INDEX_OUTPUT for writing
    try:
        index = open(INDEX_OUTPUT, 'w')
    except Exception as e:
        print(f'error: the index file could not be opened for writing: ', e)
        sys.exit(10)

    # print out the header
    for l in list(header_lines):
        l.rstrip()
        index.writelines(l)

    # end the header with a markdown line and line break
    write_md_line(index)

    # generate Categories based on the toplevel directories
    generate_categories(index, '.')

    # traverse the folders, writes the index to the file
    traverse_folders(index, '.', 0)

    # write the count of files
    index.write(f'\n___{total_md_files_indexed}___ files indexed\n')

    # end the index with a markdown line and extra line break
    write_md_line(index)
    index.write('\n')

    # print out the footer
    for l in list(footer_lines):
        l.rstrip()
        index.writelines(l)

    # close out the index file
    index.close()


######################################################################################
#
# Supporting functions
#

def traverse_folders(f, rootfolder, level):
    
    global total_md_files_indexed

    try:
        folders = sorted([d.path.lstrip('./') for d in os.scandir(rootfolder) if d.is_dir() and not d.name.startswith('.')])
    except Exception as e:
        print(f'error: could not traverse folder {rootfolder}: ', e)
        return []
    
    for folder in list(folders):
        total_md_files_indexed += generate_md_file_index(f, folder, level)
        folders.extend(traverse_folders(f, folder, level + 1))

    return folders

def generate_md_file_index(f, folder, level):
    
    num_md_files_indexed = 0

    basename_directory = os.path.basename(folder)
    friendly_dirname = basename_directory.capitalize()

    # write the markdown header
    header = '\n###' + ('#' * level)

    f.write(f'{header} {friendly_dirname}\n')

    try:
        md_files = sorted([f.path for f in os.scandir(folder) if f.is_file()])
    except Exception as e:
        print(f'error: could not get markdown files from folder: ', e)
        return 0

    for md_file in list(md_files):
        # make sure it's actually a markdown file ending in .md, otherwise skip
        if is_markdown_file(md_file):
            # increment count of files
            num_md_files_indexed += 1
            with open(md_file) as md:
                title = md.readline()
                title = title.lstrip('# ')
                title = title.rstrip()
                f.write(f'- [{title}]({md_file})\n')
        else:
            next

    return num_md_files_indexed

def generate_categories(f, dirname):

    categories = sorted([d.name for d in os.scandir(dirname) if d.is_dir() and not d.name.startswith('.')])

    # header
    f.write(f'\n### Categories\n\n')

    for category in list(categories):
        category = os.path.basename(category)
        friendly_name = category.capitalize()
        f.write(f'* [{friendly_name}](#{category})\n')

    f.write('\n')

def create_category_path(category):
    # remove leading characters
    category = re.sub('^[^A-Za-z0-9]+', '', category)
    # remove trailing characters
    category = re.sub('[^A-Za-z0-9]+$', '', category)
    # remove anything else which isn't a alphanumeric or '/' character in the category string)
    category = re.sub('[^A-Za-z0-9\/\-]+', '', category)

    # now create the directory if it isn't there
    Path(category).mkdir(parents=True, exist_ok=True)

    return(category)

#
# Sanitzation and check functions
#
def title_to_filename(title_string):
    words = title_string.split()
    filename = ''
    for word in words:
        # remove any non-Alphanumeric characters
        word = re.sub('[^A-Za-z0-9]+', '', word)
        # concatenate into hypenated string
        filename += word.lower() + '-'
    
    # strip the last '-' on the filename
    filename = filename[:-1]
    filename += '.md'
    return filename

def capitalize_title_words(title_string):
    return title_string.title()

def is_markdown_file(md_file):
    filename, file_extension = os.path.splitext(md_file)
    if file_extension == '.md':
        return True
    else:
        return False

def write_md_line(f):
    f.write(f'\n---\n')

def readfile(filename):
    try:
        f = open(filename, "r")
        contents = f.readlines()
    except:
        contents = ""
    
    return contents


#
#
######################################################################################
    
 #
 # Standard call to main()
 #   
if __name__ == "__main__":
    main()
