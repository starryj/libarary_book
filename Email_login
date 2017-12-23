import pymongo
import smtplib
import poplib
import random
import requests
from email.mime.text import MIMEText
from email.utils import formataddr
from email.mime.multipart import MIMEMultipart
from email.mime.image import MIMEImage
import re
pophost = 'pop.sina.com'
smtp_host = 'smtp.sina.com'
port = 25
from_mail = 'xxxx@sina.com'
password = 'xxxxxxxxxxx'


class send_mail(object):
    def __init__(self, subject, content, href, to_mail):
        self.to_mail = to_mail
        self.book_name = subject
        self.content = content
        self.href = href
        self.send = False
        self.html = """
        <p>有书了呦！</p>
        <p><a href={href}>{book_name}</a></p>
        <p>{content}</p>
        <p><img src="cid:image1"></p>
        """
        client = pymongo.MongoClient('localhost')
        db = client['表情包']
        self.coll = db['image_hrefs']

    def my_mail(self):
        try:
            T = 1
            msg = MIMEMultipart('related')
            msg['From'] = formataddr(['BOOK_ROBOT', from_mail])
            msg['To'] = formataddr(['book_info', self.to_mail])
            msg['Subject'] = self.book_name
            msgAlternative = MIMEMultipart('alternative')
            msg.attach(msgAlternative)
            msg.attach(MIMEText(self.html.format(href=self.href, book_name=self.book_name, content=self.content), 'html', 'utf-8'))
            while T:
                try:
                    sum_images = self.coll.count()
                    random_nub = random.choice(range(sum_images))
                    href = self.coll.find_one({'_id': random_nub})['href']
                    self.coll.remove({'_id': random_nub})
                    res = requests.get(href)
                    msgImage = MIMEImage(res.content)
                    T = 0
                except:
                    continue
            msgImage.add_header('Content-ID', '<image1>')
            msg.attach(msgImage)
            server = smtplib.SMTP(smtp_host, port)
            server.login(from_mail, password)
            server.sendmail(from_mail, self.to_mail, msg.as_string())
            server.quit()
            self.send = True
        except smtplib.SMTPException:
            self.send = False
        return self.send

    def send_message(self, title):
        while not self.send:
            serch = poplib.POP3_SSL(pophost)
            serch.user(from_mail)
            serch.pass_(password)
            list = len(serch.list()[1])
            print(list)
            try:
                for i in range(int(list)-22, int(list)):
                    f = serch.retr(i)
                    subject = re.search(r"'Subject:(.*?)'", str(f), re.S).group(1)
                    if title != subject:
                        self.my_mail()
                        print('send mail')
                        break
            except:
                for i in range(int(list)-2, int(list)):
                    f = serch.retr(i)
                    subject = re.search(r"'Subject:(.*?)'", str(f), re.S).group(1)
                    if title != subject:
                        self.my_mail()
                        print('send mail')
                        break

if __name__ == '__main__':
    my_mail = send_mail('python', '没什么，就随便看看', 'http://www.baidu.com', 'xxxxxxxx@qq.com')
    my_mail.send_message('python')
