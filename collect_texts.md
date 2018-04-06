
### Get all author links


```python
import requests, re
from bs4 import BeautifulSoup
from IPython.display import HTML, display

home = "http://latin.packhum.org/browse" # page with all author links

def get_links(soup):
    
    for tag in soup.findAll('a'): # find tags that contain links
        link = tag['href']
        match = re.findall('/author/[0-9]+', link) # find all links in this format
        if match:
    
            np_link = 'http://latin.packhum.org' + match[0] # create whole link
            
            print(np_link)
            
            author_name = tag.findAll('span')
            author_name = author_name[0].text # get author name
        
            if np_link not in author_links:
                author_links.append([author_name, np_link])
        
```


```python
author_links = [] # each element: [author name, author link]

req = requests.get("http://latin.packhum.org/browse") # the page where we search for author links
soup = BeautifulSoup(req.text, 'lxml')

get_links(soup) # calls function to collect all author links

print(len(author_links))
```

    362


### Get all text links (to page 1) from "author_links"


```python
def get_text_links(soup, author_link):
    
    for tag in soup.findAll('a'):
        link = tag['href']
        match = re.findall('/loc/.*$', link) # finds all links to texts
        if match:
        
            text_name = tag.findAll('span')
            text_name = text_name[1].text
            
            print(match[0] + ' ' + author_name + ' ' + text_name)
            
            np_link = 'http://latin.packhum.org' + match[0] # create whole link
            print(np_link)
            
            if np_link not in text_links:
                text_links.append([author_name, text_name, np_link])

```


```python
text_links = [] # each element: [author name, text name, link to page 1]

for i in author_links:
    author_name = i[0]
    author_link = i[1]
    req = requests.get(author_link) # the page where we search for text links
    soup = BeautifulSoup(req.text, 'lxml')
    
    get_text_links(soup, author_link) # calls function to collect all links to texts
    
print(len(text_links))
```

    836


### Get all texts (every page) from "text_links"


```python
def get_text(page_link): # returns text from current page
    
    req = requests.get(page_link)
    soup = BeautifulSoup(req.text, 'lxml') # current page soup
    
    page_text = '' # this is needed. if it remains empty, we know we reached the end of the text pages
        
    for tag in soup.findAll('table'):

        for tr in tag.findAll('tr'): # find all tags with text
            line = tr.text

            while '\n' in line: # delete empty lines 
                line = line.replace('\n', '')

            while '                           ' in line: # delete extra spaces before numeration
                line = line.replace('                           ', '\t\t')
            line = line.lstrip() # delete spaces from beginning of string
            line = line.rstrip() # delete spaces from end of string
            
            page_text += line
            page_text += '\n'
            
    return page_text

def get_whole_text(author, text_name, text_link): # finds each page of a text, compiles complete text
    
    file_name = author + ' - ' + text_name + '.txt' # unique name for text    
    f = open('text files/' + file_name, 'w', encoding = 'utf-8') # creates new file for a text
    
    # all files will be saved in 'text files' folder
    # you need to create this folder first
    
    print(author + ' - ' + text_name)
    
    whole_text = ''
   
    template = text_link[:-1]
    index = 0
    
    current_page = template + str(index) # 1st page link
    print(current_page)

    page_text = get_text(current_page) # get text from 1st page
    whole_text += page_text # add page text to total text

    while page_text: # keep getting text if next page exists

        index += 1
        current_page = template + str(index) # link to next page
        print(current_page)

        page_text = get_text(current_page) # get text from next page (will be empty if there's no next page)
        whole_text += page_text
            
    f.write(whole_text)          
    f.close()
    
    all_texts.append([author, text_name, whole_text])

```


```python
all_texts = [] # each element: [author, text name, text]

for item in text_links: # compile every text
    author = item[0]
    text_name = item[1]
    text_link = item[2]
    
    get_whole_text(author, text_name, text_link) # creates new file for each text
                                                 # the texts are now also in 'all_texts'

print(len(all_texts))
```

    836


### Save every text into "all_texts.txt"


```python
f1 = open('all_texts.txt', 'w', encoding = 'utf-8') # new file for all texts

for item in all_texts:
    author = item[0]
    text_name = item[1]
    whole_text = item[2]
    
    f1.write(author)
    f1.write('\n')
    f1.write(text_name)
    f1.write('\n\n')
    f1.write(whole_text)
    f1.write('-----------------------------------------------')
    f1.write('\n')
    
f1.close()
```

### Open text files, load into array


```python
import glob
import os

texts = [] # each element is [author and text name, text]

path = 'text files/' # path to folder with texts

for filename in glob.glob(os.path.join(path, '*.txt')): # finds all .txt in 'text files' folder

    f = open(filename, 'r', encoding = 'utf-8')
    text = f.read()
    
    texts.append([filename, text])
    
    f.close()
    
print(len(texts))
```

    836

