Javascript security analysis (JSA) is a program for javascript analysis during web application security assessment.

# Capabilities of jsa.py:

- Looking for js files inside the first, second and third level js files. For example, http(s)://host.com/file.js contains a string "https://host.com/file.js" which could be different from already known 1st level js files and contain additional endpoints or secrets.
- Displaying endpoints for the second and third level js files.
- It modifies found endpoins in the javascript file from /endpoint to http(s)://host_from_js_file.com/endpoint. This is a very usefull approach when you have a huge list of javascript files and would like to collect a list of all URLS.
- Excludes and prints 3rd party js files like https://googleapis.com or //facebook.net (most likely 2nd level js file) to reduce script runtime and remove unnecessary endpoints. Also, it is usefull to identify 3rd party js files since we can expand our attack surface and exploit vulnerabilities on a 3rd party website to try to make change to the javascript file.
- Checks the http code of the js file using HEAD method (it's faster than just GET). If 200 - it goes for further processing (because all js files should have a 200 code, otherwise there will be no loading on the page), if 404 - then such a code indicates a non-existing file (or an unregistered host with a file) that can be uploaded by an attacker for the displaying on the target page.
- Remove duplicates - js files and endpoints. By default, most of the tools for js grabbing (like subjs, gau, etc) can provide a list of js files containing duplicates. Even if they performed deduplication procedure, a list can still contain duplicates since, for example, http(s)://host.com/file.js and http(s)://host.com/file.js?identifier=random_str are the same js files. Deleting duplicates can greatly boost the program's performance.
- Remove unnecessary files with such extensions .css|.png|.jpg|.svg|.jpeg|.ico|.gif|.woff|.woff2|.swf.

# Capabilities of automation.sh:

- Searching for js files for provided host (http(s)://host.tld) in stdin using:
  - Wayback Machine (GAU), - I'm using mimetype:(application|text)/javascript to reduce execution time;
  - Crawling (hackrawler);
  - subjs tool;
  - GitHub search (github.py).
- Extraction of js files from js files.
- Printing endpoints.
- Processing Wayback Machine output and separating 200 OK js files from non-existent. If a file is deleted on the host, the script automation/./404_js_wayback.sh will save a link in the way https://web.archive.org/web/20060102150405if_/$url for credentials checking.
- Checking for credentials leakage using nuclei and a custom template containing 957 regexes on:
  - Alive js files (200 OK on the target host);
  - A copy of deleted js files from Wayback Machine.
- Parameters discovery on found endpoints using Arjun.

# Usage & installation for jsa.py:
git clone https://github.com/w9w/js_extractor.git && cd js_extractor
echo "https://host.com/file.js" | python3 js_extractor.py

Example for pulling out js files and processing:
echo "https://subdomain.host.com" | subjs | python3 js_extractor.py

# Usage for massive and parallel scanning (~lightning-fast execution):

cat ~/lists/domains/host.com/http_s_hosts.txt | subjs | parallel -j 20 'echo "{}" | python3 /root/js_extract.py'.

You can get parallel GNU here https://www.gnu.org/software/parallel/. Don't forget to delete that annoying message.

# Usage & installation for automation.sh:

chmod +x installation.sh
./installation.sh
echo "http(s)://host.com" | ./automation.sh

# Existing js files' analysis solutions:
- **https://github.com/GerbenJavado/LinkFinder** - extracts relative URLs and some full URLs with http(s) prefix;
- **https://portswigger.net/bappstore/0e61c786db0c4ac787a08c4516d52ccf** - same but can't be used for massive processing of js files;
- **https://github.com/KathanP19/JSFScan.sh** - also relatives paths only but in addition these features:
  - js discovering using gau on all the links and subjs;
  - searching for secrets using SecretFinder (I was thinking a long time about SecretFinder vs nuclei and I found out that SecretFinder is not that reliable and it returns a lot of useless results even with improvements so I picked nuclei);
  - printing variables and words.
  
### Roadmap:

- ✅ replace \[]// with http(s) host.tld /, if it exists;
- ⬜️ perform a check on every <script> part in the html page
- ✅ deletion of duplicate files of the second level in relation to the files of the first level;
- ✅ setting the js file in the parameter when calling the program, still saving stdin;
- ✅ output the second level js files **optionally**, by parameter;
- ⬜️ save all found endpoints to a file **optionally**, by parameter;
- ⬜️ save all found deep-level js files to a file **optionally**, by parameter;
- ✅ improve the exclusion of 3rd party scripts by domain for multiple domains during bulk scanning, if possible;
- ✅ define domain and tld using re depending on line, if it's possible;
- ⬜️ check available HTTP methods for endpoints;
- ⬜️ retire js check via downloading js files to the temporary directory using wget;
- ✅ credentials leak check using secretfinder.py with extended regular expressions;
- ✅ brute-forcing parameters for endpoints using arjun.py;
- ⬜️ make a file with endpoints along with parameters for pipelining to check for XSS'es, CORS misconfigs, etc.
- ⬜️ identificate and process .map files;
- ⬜️ rewrite a tool in Golang;
- ⬜️ multithreading, - only in Golang (multithreading in Python is terrible imo).

Special thanks to these beautiful people from whom I  ̶s̶h̶a̶m̶e̶f̶u̶l̶l̶y̶ ̶s̶t̶e̶a̶l̶e̶d̶ borrowed some tools for automation.sh :D :

Corben Leo @lc for github.com/lc/subjs and github.com/lc/gau;
Luke Stephens @hakluke for github.com/hakluke/hakrawler;
Gwendal Le Coguic @gwen001 for https://github.com/gwen001/github-search/raw/master/github-endpoints.py;
Project discovery @projectdiscovery for github.com/projectdiscovery/nuclei and github.com/projectdiscovery/httpx;
Somdev Sangwan @s0md3v for https://github.com/s0md3v/Arjun (I needed to fork it for automation ease).


# Ways to contribute

- Suggest a useful feature
- Report a bug
- Fix something and open a pull request
- Create a burp suite plugin
- Spread the word
