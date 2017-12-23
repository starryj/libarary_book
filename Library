import requests
from bs4 import BeautifulSoup
import re
import pymongo
import time
from Email_Login import send_mail
from apscheduler.schedulers.blocking import BlockingScheduler


class Book(object):
    def __init__(self):
        mongo_uri = 'localhost'
        databases = '图书馆'
        client = pymongo.MongoClient(mongo_uri)
        self.db = client[databases]
        self.coll = '图书'
        self.books = dict()
        self.url = "http://huiwen.ujs.edu.cn:8080/opac/openlink.php?strSearc" \
                   "hType=title&match_flag=forward&historyCount=1" \
                   "&strText={name}&doctype=ALL&with_ebook=on&displaypg=20&s" \
                   "howmode=list&sort=CATA_DATE&orderby=desc&dept=ALL"

    def get_book_info(self, book_name):
        header = {'Host': 'huiwen.ujs.edu.cn:8080',
                  'User-Agent': 'Mozilla/5.0 (Windows NT 6.1; WOW64; rv:52.0) Gecko/20100101 Firefox/52.0',
                  'Accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8',
                  'Accept-Language': 'zh-CN,zh;q=0.8,en-US;q=0.5,en;q=0.3',
                  'Accept-Encoding': 'gzip, deflate'
                  }

        req = requests.get(self.url.format(name=book_name), headers=header)
        req.encoding = 'utf-8'
        html = req.text
        soup = BeautifulSoup(html, 'lxml')
        pattern = r'\d+'
        book_list_info = soup.findAll('li', {'class': 'book_list_info'})
        for book_info in book_list_info:
            book_name_and_index = book_info.h3.get_text().strip()
            book_s = book_info.p.get_text().strip().replace(' ', '')
            numb = re.findall(pattern, book_s)
            if numb[0] == numb[1]:
                n = numb[0]
                split_book = re.split(str(n), book_s, 2)
                book_detail0 = split_book[0].strip().replace('\t', '').replace(' ', '') + str(n)
                book_detail1 = split_book[1].strip().replace('\t', '') + str(n)
                book_detail2 = split_book[2].strip().replace('\t', '').replace(r'(0)馆藏', '')
                book_detail = book_detail0 + '\n' + book_detail1 + '\n' + book_detail2
            else:
                n = numb[1]
                split_book = re.split(str(n), book_s, 1)
                book_detail0 = split_book[0].strip().replace('\t', '').replace(' ', '') + str(n)
                book_detail1 = split_book[1].strip().replace('\t', '').replace(r'(0)馆藏', '')
                book_detail = book_detail0 + '\n' + book_detail1
            print(book_name_and_index)
            print(book_detail)
            books_detail = book_name_and_index + book_detail
            href = 'http://huiwen.ujs.edu.cn:8080/opac/' + book_info.a['href']
            if href not in self.books:
                assert isinstance(books_detail, str)
                self.books[href] = books_detail
        print(self.books)
        return self.books

    def up_data(self, book_name):

        collection = self.db[self.coll]
        if not collection.find_one({'_id': book_name + str(0)}) and not collection.find_one({'_id': book_name + str(1)}):
            collection.insert_one({'_id': book_name + str(0)})
        datas = self.get_book_info(book_name)
        for i in datas:
            if not collection.find_one({'href': i}) and collection.find_one({'_id': book_name + str(1)}):
                pattern = r'可借复本：?\d+'
                can = re.search(pattern, datas[i]).group(0)
                mail = send_mail(subject=book_name + can, content=datas[i], href=i, to_mail='276694730@qq.com')
                mail.send_message(book_name + can)
                print('send')
                time.sleep(2)
                mail = send_mail(subject=book_name, content=datas[i], href=i, to_mail='hexiyuchi@sina.com')
                mail.send_message(book_name + can)
                time.sleep(2)
            collection.update({'href': datas[i]}, dict(href=i, content=datas[i], timestamp=time.ctime()), True)
        collection.remove({'_id': book_name + str(0)})
        try:
            collection.insert_one({'_id': book_name + str(1)})
        except:
            pass
        print('insert')

    def search_book(self, detail_book_name):
        pattern = r'可借复本：?\d+'
        pa = r'\d+'
        data = self.get_book_info(detail_book_name)

        for i in data:
            nums = data[i]
            num = re.search(pattern, nums).group(0)
            if int(re.search(pa, num).group(0)) != 0:
                mail = send_mail(subject=detail_book_name, content=data, href=data.keys(), to_mail='276694730@qq.com')
                mail.send_message(detail_book_name + num)
                print('OK')
                mail = send_mail(subject=detail_book_name, content=data, href=data.keys(), to_mail='hexiyuchi@sina.com')
                mail.send_message(detail_book_name + num)

if __name__ == '__main__':
    one_book_name = 'python算法教程'
    book_search_name = 'python'
    book_search = Book()
    sche = BlockingScheduler()
    sche.add_job(func=book_search.up_data, trigger='cron', day='*', hour='18,19,20', args=(book_search_name,))
    sche.add_job(func=book_search.search_book, trigger='cron', day='*', hour='18,19,20', args=(one_book_name,))
    sche.start()
