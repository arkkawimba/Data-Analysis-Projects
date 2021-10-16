import mysql.connector as mysql
import urllib.request, urllib.parse, urllib.error
import string
from bs4 import BeautifulSoup
from random import randrange
import re
import time

start_time = time.time()

db = mysql.connect(
    host = "localhost",
    user = "root",
    passwd = "admin",
    database = 'python'
)

cursor = db.cursor()

def bookTitle(x): #checking the book title from the website
    title = x.find('meta',property='og:title')
    return title['content']

def checkLanguage(x): #checking the book language. This script only intended to scrape English books
    tdTag = x('td')
    for tag in tdTag:
        if tag.string == 'English':
            return 'English'
            break
    return 'Not English'

def cleaningTxt(x,y):
    mystr = [line.decode() for line in fhand] #decoding to UTF-8
    mystr = '\t'.join([line.strip() for line in mystr]) #stripping the new line, making the entire text 1 single very long line

    
    excludePunctuation = set(string.punctuation)
    replaceWith=' '
    mystr = ''.join([replaceWith if i in excludePunctuation else i for i in mystr])
    words = mystr.split()#splitting each words

    #removing numbers and other excluded words
    excludeWords = ['gutenberg','org','com','www','http','https']
    for i in list(words):
        if i.isdigit() is True:
            words.remove(i)
        elif i in excludeWords:
            words.remove(i)
        elif any(y.isdigit() for y in i):
            words.remove(i)
        elif i.startswith(tuple(string.ascii_letters)) is False:
            words.remove(i)
            
    #turning all words into lowercase
    for i in range(len(words)):
        words[i] = words[i].lower()

    #adding new words to dictionary
    for i in words:
        y[i] = y.get(i,0)+1 #checking if the word i exist or not. If not, it add i as new word. If yes, it adds the count by 1

    return y

def addData(x,y): #adding the book id and title to MySQL table
    query = ('INSERT INTO books (id,title) VALUES (%s, %s)')
    values = (x,y)
    cursor.execute(query,values)
    db.commit()

def addWordCount(x):
    for word,count in x.items():
        try: #checking if the word already exist or no
            query = ('INSERT INTO words (word,count) VALUES (%s,%s)')
            values = (word,count)
            cursor.execute(query,values)
            db.commit()
        except: #adding the word count to existing word
            query = ('UPDATE words SET count = count + %s WHERE word= %s')
            values = (count,word)
            cursor.execute(query,values)
            db.commit()

def checkBookID(x): #checking whether this book have been scraped before or not
    cursor.execute('SELECT id FROM books')
    dbBookID=cursor.fetchall()
    for i in dbBookID:
        cleanID = re.findall('\d+',str(i))
        if str(x) == cleanID[0]:
            return 1
    return 0

wordCount={}
bookCount=1
trialCount=0

print('WELCOME')
print('How many books would you like to scrape this session?')
maxBookCount = int(input('Please enter a number: '))

while bookCount<=maxBookCount:
    bookID=randrange(0,50000)
    trialCount+=1
    print('========================================================================')
    print('Trial: ',trialCount)
    print('Book number ', bookCount)
    print('Book ID: ',bookID)
    
    if checkBookID(bookID)==1:
        print('Book have been scraped before. Finding new book.')
        continue
    else:
        print('New book. Scraping continues')
    
    url = 'https://www.gutenberg.org/ebooks/'+str(bookID)

    try:
        html = urllib.request.urlopen(url).read()
    except:
        print('Error reaching book page. Finding new book')
        continue

    soup = BeautifulSoup(html, 'html.parser')

    
    print('BOOK FOUND')
    print('Title: ',bookTitle(soup))
    print('Language: ',checkLanguage(soup))

    if checkLanguage(soup)=='Not English':
        print('Book is not in English. Finding new book')
        continue

    txtUrl = 'https://www.gutenberg.org/cache/epub/'+str(bookID)+'/pg'+str(bookID)+'.txt'
    print(txtUrl)
    
    try:
        fhand = urllib.request.urlopen(txtUrl)
    except:
        print('Error in opening book text file. Finding new book')
        continue
    
    print('Scraping successful')
    addData(bookID,bookTitle(soup))
    wordCount=(cleaningTxt(fhand,wordCount))
    bookCount+=1

scrapedWords=0
sumScrapedWords=0

for x,y in wordCount.items():
    scrapedWords+=1
    sumScrapedWords = sumScrapedWords + y

addWordCount(wordCount)
print('\n++++++++++++++++++++++++++++++++++++++++++++++++++++++++++')
print(time.time()-start_time,'seconds')
print('Scraping session finished')
print(bookCount-1,'books scraped')
print(sumScrapedWords,'total words')
print(scrapedWords,'unique words found')
