# Introduction
This tool is used for (almost) automatic migration of Confluence documentation into Discource.

It handles:
- Html to markup formatting conversion (many thanks to `confluence-to-markdown` tool)
- Functioning links between topics
- Images and attachments uploads

As an example data documentation from https://confluence.origam.com is used (site is not working anymore).

# Installation
- Install Python: https://www.python.org/
- Install pandoc: https://pandoc.org/installing.html
- Export Confluence documentation in html format (if you wish to import other Confluence data than our example)
- FYI, https://www.npmjs.com/package/confluence-to-markdown is already downloaded in this repository, so no need to install it


# Migration process steps
Working directory for whole migration is `node_modules/confluence-to-markdown`, here everything happens.
(To be more clean, my solution should've been created in separate directory and not been mixed with original `confluence-to-markdown` directory...I know...)

Migration steps along with directories description:
  1. See `source_all` - exported .html files (with images and attachments) from Confluence. This is our entry data. Copy you prospective files from Confluence export here.
  2. See `python` - these are Python scripts which are executed in further .bat files. No need to change them now.
  3. Run `0_prepareHtmls.bat` - copies .htmls from `source_all` to `source` and does some preprocessing in these files
  4. See `source` - preprocessed .html files are now here
  5. Run `1_createMDs.bat` - transforms .html files from `source` to .md files in `import`
  6. See `import` - .md files
  7. Preparation for images and attachments automated upload (This is the most complicated part):
      1. For our example export I have prepared files in `upload` directory, which we copied from Confluence export
      2. For further steps excel `excel/Discourse files (PROD).xlsx` will be used (or `...(LOCAL).xlsx`, if you're working on testing/local environment)
      3. Copy all file names from `upload` directory, sorted by file size descending **!** (to cover duplicities, see later), into this excel into sheet `all files`, column `B` (`FileName`)
      4. Now we need to manually upload all files from `upload` directory into Discourse . To do this, create any topic in Discourse forum, then manually drag all files in batches of 20 files into the topic text. Ask Discourse admin to set that max. 20 files can be concurrently uploaded (it's max. value).
      5. During upload to Discourse some errors concerning duplicate files can be shown (it seems that you can't upload one file under different names multiple times to Discourse). Ignore errors for now.
      6. Copy generated text from Discourse topic into excel into sheet `translation`, column `C` (`Discourse text`)
      7. Now for handling files duplicities. I handled them in excel in sheet `all files`, formula in column `F` (`FinalFileNameBase`) - when `FileNameBase` cannot be looked up, then file name from previous row is used (I assume it's file with same content, because in step 3 we entered files by descending size. But better check each lookup error by yourself, so correct replacements in column `F` are generated)
      8. If correctly filled, excel now produces needed Python code in sheet `all files`, column `J` (`Final (md)`) - `_newFileContent = re.sub...etc.`. Copy/replace all column into file `python/prepareMDs.py` (in section `# images and attachments`, LOCAL or PROD environment)
     5. The result of all these steps are uploaded images and files in Discourse forum and prepared python script `python/prepareMDs.py`
     6. Maybe I forgot some steps here, in case of need contact me directly and we will workthrough your case, and then I'll update this readme.
  8. Run `2_prepareMDs_PROD.bat` (or `2_prepareMDs_LOCAL.bat`) - preprocesses .md files in directory `import`
  9. See `import` - now preprocessed .md files are here
  10. Run `3_runImport_PROD.bat` (or `3_runImport_LOCAL.bat`) - finally import topics from prepared .md files. Before executing this please explore script `python/runImport.py` to understand it and set needed variables (see sections `# set >>>>>>`...). I recommend to comment the execution of `createAllTopics` during the first run to test the connection etc.
  11. For import to successfully execute, some Discourse settings will need to be set (eg. minimum topic title length, max. post length, etc. - just see what errors arise during migration), together with API throughput settings  (see eg. https://meta.discourse.org/t/available-settings-for-global-rate-limits-and-throttling/78612)
  12. After finishing the import script all processed .md files are moved into `import_DONE` folder.

FYI, directories `bin`, `src` and `test` are not my creation, they belong to downloaded `confluence-to-markdown` tool. I just slightly changed these files for use of the migration (topic names generation etc.): `src/Formatter.coffee`, `src/Page.coffee`, `src/Utils.coffee`

To run again the migration, repeat all steps (or just from step 4 onward if you're importing same .html files once again).

Good luck.

# Known shortcomings
There are few things this tool doesn't handle:
- Some special characters (like `.`, `=`) are not included in generated Discourse topic titles 
- Links to chapters inside topics (anchors) are not working (can be solved by adding `<div data-theme-toc="true"></div>` in topic text)
- No hierarchy tree in Discourse topics is generated, just flat structure of topics

# Author
Martin Zákostelský
martin.zakostelsky@seznam.cz
