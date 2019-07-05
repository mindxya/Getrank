import requests,re,threading,gevent,socket,time,os
from bs4 import BeautifulSoup
from fake_useragent import UserAgent
from urllib import parse
import pandas

# 当前工作路径
path = os.path.split(os.path.realpath(__file__))[0]

# 搜狗手机版排名查询
# 参数 域名，关键字，查询页数
class Sogoum:

    def __init__(self,domain,keywords,pn):
        self.domain = domain
        self.keywords = keywords
        self.pn = pn
        self.info = {}
        self.info['域名'] = domain
        self.info['关键字'] = keywords

    def Get_MsogouHtml(self):
        ua = UserAgent()
        faq_head = {
            "User-Agent": ua.random
        }

        chu = os.path.exists(path+"\\domain\\"+self.domain+self.keywords+".txt")
        if chu == False:
            chu = 0
        else:
            f = open(path+"\\domain\\"+self.domain+self.keywords+".txt")
            chu = f.read()
            f.close()

        chushipn = 0
        for ii in range(0, self.pn):
            chushipn = chushipn + 1
            #print("当前查询第%s页" % chushipn)
            try:
                url = "https://m.sogou.com/web/searchList.jsp?keyword="+ parse.quote(self.keywords) +"&p="+str(ii)
                bhtml = requests.get(url, headers=faq_head)
                btxt = bhtml.content.decode("utf-8")
                bcode = bhtml.status_code
                self.baiduhtml = btxt
                if bcode == 200:
                    pm = self.Msogoupm()
                    if pm != None:
                        if chu == 0:
                            self.info['初始化排名'] = chu
                            yema = chushipn - 1
                            weizhi = pm
                            dangqian = str(int(yema * 10) + weizhi)
                            self.info['当前排名'] = dangqian
                            biaoshi = 0
                            return self.info,biaoshi
                        else:
                            biaoshi = 1
                            yema = chushipn -1
                            weizhi = pm
                            dangqian = str(int(yema * 10) + weizhi)
                            sjpm = str(int(chu) - int(dangqian))
                            self.info['初始化排名'] = chu
                            self.info['当前排名'] = dangqian
                            self.info['升降情况'] = sjpm
                            return self.info,biaoshi

                       #return chushipn -1, pm,chu
            except Exception as e:
                print("链接百度超时错误为：", e)
                return None

    def Msogoupm(self):
        soup = BeautifulSoup(self.baiduhtml, 'html.parser')
        try:
            # 筛选出含有result类的div，因为这是只包含搜索结果列表的标签数组
            p = re.compile(r'^citeurl')
            # 筛选出含有该域名的列表，并算出排名
            str = re.compile(r'%s' % self.domain)
            # 找到所有的搜索结果列表
            contentItem = soup.find(class_="results").find_all('div', p)
            #print(contentItem)
            for index, list in enumerate(contentItem):
                _list = ''.join(re.split(r'\s+', list.get_text()))
                #print(_list)
                if self.domain in _list:
                    return index
        except Exception as e:
            #print("当前页未找到排名")
            return None


def save(contents,filename):
    fh = open(path+"/"+filename, 'a+', encoding='utf-8')
    fh.write(contents+"\n")
    fh.close()

def run():
    xlsa = []
    f = open("list.txt")
    for line in f.readlines():
        curLine = line.strip().split(" ")
        lista = curLine[0].split('|')
        p = 5
        y = lista[0]
        k = lista[1]
        bpc = Sogoum(y, k, p)
        rss = bpc.Get_MsogouHtml()
        if rss != None:
            xinxi, biao = rss
            if biao == 0:
                print('初始化', xinxi['当前排名'])
                f1 = open(path + "\\domain\\" + y+k + ".txt", 'w')
                f1.write(xinxi['当前排名'])
                f1.close()
                print("排名结果：第%s位" % xinxi['当前排名'])
            else:
                chua = xinxi['初始化排名']
                dangqian = xinxi['当前排名']
                sjpm = xinxi['升降情况']
                print("排名结果：初始化排名:%s位,当前排名:%s位,升降情况:%s位" % (chua, dangqian, sjpm))
                xlsa.append(xinxi)
        else:
            print("排名结果:不在前5页范围。")
    if len(xlsa) > 0:
        file_name = str(time.strftime("%Y%m%d%H%M%S", time.localtime())) + ".xlsx"
        df = pandas.DataFrame(xlsa)
        df.to_excel(file_name)

if __name__ == '__main__':
    run()
