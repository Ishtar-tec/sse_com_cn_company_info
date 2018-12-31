## 从上海证券交易所爬取沪股公司近几年内的基本数据及财务数据
#### 前言
###### 闲聊
 + 金融市场很大程度上是一个大数据世界
 + 怀揣啊着"天下兴亡，匹夫有责"的书生意气
 + 来心忧一下祖国的经济晴雨表吧
 + 索罗斯很厉害，但他那种宏观面的数据之后爬吧
 + 王亚伟，徐翔也贼强，但他们那种信息更多是**实地考察**与**社会工程**的趋于完美的结合，爬数据完全不沾边的
 + 巴神说，要重点考察基本面，别老盯着涨跌
 + 我不知道巴神的理念对不对，但他老人家名扬世界，而且他那种数据与大数据搭界, 于是我家爬虫也来赶赶基本面的热闹
###### 甄选方案
 + 到chrome的开发者工具network列表区过滤ajax请求
    - 发现了请求返回目标数据的ajax请求
        * url: http://listxbrl.sse.com.cn/companyInfo/showmap.do
        * post data: {'stock_id': 60000, 'report_year': 2017, 'report_period_id': 5000}
    - 可是呀，这个json可读性性差得要哭，还没哭（下边附图）
    - 当然了，拿来与页面核对一下，就知道了，但是那会增加不确定性
        * 浏览器加载json前要不要改值呀
        * json重的数据是不是每次都是按原顺序渲染到表格呀
        * ...
    - 读一下js就知道了啊，这个好像是的，但如果有更好的选择的话，谁会去读那种压缩过了，只剩类似a,b,aa,bb,_1,_1a命名套路的混淆代码呢？
    - 最省事儿的办法，当然还是用python控制selenium像其他用户的渲染流程一样渲染，然后取值啊
    <!--![上交所财ajax返回的报表格数据json可读性太差](http://pk6q1v5l2.bkt.clouddn.com/%E4%B8%8A%E4%BA%A4%E6%89%80%E8%B4%A2ajax%E8%BF%94%E5%9B%9E%E7%9A%84%E6%8A%A5%E8%A1%A8%E6%A0%BC%E6%95%B0%E6%8D%AEjson%E5%8F%AF%E8%AF%BB%E6%80%A7%E5%A4%AA%E5%B7%AE.png)-->

<div style="text-align:center">
 <img alt="上交所财ajax返回的报表格数据json可读性太差" style="border:1px solid #ccc; display: block; margin: 20px auto 80px" src="http://pk6q1v5l2.bkt.clouddn.com/%E4%B8%8A%E4%BA%A4%E6%89%80%E8%B4%A2ajax%E8%BF%94%E5%9B%9E%E7%9A%84%E6%8A%A5%E8%A1%A8%E6%A0%BC%E6%95%B0%E6%8D%AEjson%E5%8F%AF%E8%AF%BB%E6%80%A7%E5%A4%AA%E5%B7%AE.png">
</div>

 + 确定了用selenium，那么要不要用框架呢？
    - 我要爬取的是上市公司的基本信息和财务信息，当前表格页的url是 http://listxbrl.sse.com.cn/companyInfo/toCompanyInfo.do?stock_id=600000&report_period_id=5000
    - url参数中的stock_id更换一下就可以获得对应的公司信息了
    - 那么循环股票代码列表，selenium控制chrome请求，渲染，for循环表格提取就行了
    - 结论是不用**pyspider**和**scrapy**框架，也不需要**中间人代理**啥的
#### 代码
###### 获取全部沪股代码
        * 其实在上交所下载股票代码列表excel文件也是可以的，但我觉得写代码更省事儿
        * 之前我我爬取过http://www.askci.com获取到了所有A股的注册信息存在mongodb里了，这里写个脚本取出来写入本地文件吧，提取代码

```
# -*- coding:utf-8 -*-
import pymongo
import json
import re

client = pymongo.MongoClient('localhost')
db = client['universal']
collection = db['askci_a_stock']

stock_codes = []
st_name_reg = re.compile('\*?ST\s*')
code_reg = re.compile('/6\d{5}/')
cursor = collection.find({
    'stock_code': {'$regex': '6\d{5}'}, # 正则匹配6开头的股票代码
    'stock_name': {'$regex': '^\s*[^\*S]'}, # 正则匹配非st处理的股票名称
}).limit(10000)
for stock in cursor:
    stock_codes.append(stock['stock_code'])
    print('Appended stock_code: '+ stock['stock_code'])
print('Done!')

output_path = 'stock_codes.list.json'
with open(output_path, 'a+') as f:
    f.write(json.dumps(stock_codes, ensure_ascii=False))
    print('{0:s} has been wrote, done.'.format(output_path))
f.close()
```

###### 以上代码生成的的股票代码列表文件: shangjiaoshuo_stock_codes.list.json
```
["600000", "600004", "600006", "600008", "600012", "600016", "600018", "600007", "600019", "600010", "600009", "600015", "600020", "600022", "600023", "600029", "600030", "600031", "600017", "600021", "600025", "600026", "600011", "600033", "600027", "600037", "600036", "600038", "600028", "600048", "600050", "600039", "600051", "600052", "600035", "600054", "600055", "600053", "600057", "600058", "600056", "600061", "600060", "600062", "600059", "600063", "600064", "600067", "600068", "600066", "600069", "600071", "600070", "600073", "600072", "600077", "600076", "600075", "600078", "600079", "600081", "600082", "600084", "600083", "600085", "600090", "600088", "600086", "600080", "600089", "600095", "600094", "600120", "600116", "600113", "600119", "600096", "600093", "600117", "600114", "600097", "600098", "600100", "600099", "600101", "600121", "600103", "600108", "600107", "600106", "600109", "600111", "600122", "600112", "600125", "600127", "600104", "600105", "600110", "600123", "600126", "600128", "600132", "600133", "600135", "600136", "600138", "600131", "600141", "600139", "600130", "600137", "600143", "600155", "600146", "600148", "600156", "600159", "600161", "600163", "600167", "600160", "600166", "600169", "600170", "600168", "600162", "600178", "600172", "600173", "600177", "600185", "600179", "600184", "600189", "600186", "600192", "600176", "600187", "600191", "600195", "600188", "600196", "600199", "600200", "600203", "600207", "600215", "600212", "600216", "600190", "600219", "600206", "600208", "600197", "600201", "600210", "600211", "600220", "600221", "600230", "600223", "600227", "600225", "600229", "600232", "600231", "600213", "600233", "600218", "600226", "600222", "600217", "600241", "600236", "600235", "600242", "600239", "600243", "600240", "600237", "600246", "600251", "600248", "600249", "600257", "600250", "600252", "600261", "600258", "600256", "600255", "600259", "600267", "600260", "600262", "600270", "600266", "600269", "600272", "600271", "600276", "600278", "600279", "600273", "600281", "600284", "600268", "600277", "600280", "600282", "600285", "600283", "600288", "600290", "600295", "600287", "600291", "600292", "600293", "600299", "600297", "600306", "600298", "600307", "600300", "600309", "600302", "600310", "600311", "600308", "600303", "600312", "600315", "600313", "600305", "600316", "600319", "600318", "600317", "600320", "600322", "600323", "600329", "600327", "600326", "600325", "600332", "600330", "600331", "600336", "600335", "600338", "600328", "600337", "600333", "600343", "600339", "600345", "600348", "600350", "600346", "600340", "600358", "600353", "600352", "600354", "600355", "600359", "600356", "600360", "600361", "600365", "600362", "600367", "600368", "600363", "600366", "600369", "600370", "600377", "600371", "600372", "600375", "600373", "600376", "600378", "600379", "600381", "600380", "600383", "600382", "600386", "600385", "600388", "600389", "600391", "600351", "600392", "600396", "600390", "600387", "600393", "600395", "600403", "600400", "600410", "600406", "600405", "600409", "600398", "600419", "600420", "600415", "600416", "600422", "600418", "600425", "600429", "600428", "600426", "600435", "600433", "600438", "600449", "600444", "600446", "600452", "600455", "600456", "600436", "600439", "600459", "600448", "600458", "600460", "600463", "600461", "600466", "600462", "600467", "600470", "600468", "600475", "600476", "600469", "600477", "600480", "600478", "600481", "600482", "600479", "600485", "600483", "600487", "600486", "600488", "600489", "600490", "600496", "600493", "600497", "600499", "600500", "600491", "600495", "600502", "600498", "600501", "600503", "600505", "600506", "600509", "600507", "600510", "600508", "600512", "600513", "600511", "600516", "600517", "600515", "600519", "600520", "600518", "600521", "600523", "600522", "600525", "600527", "600526", "600528", "600529", "600531", "600530", "600533", "600538", "600535", "600537", "600536", "600540", "600532", "600543", "600546", "600545", "600548", "600549", "600551", "600547", "600555", "600552", "600557", "600559", "600560", "600562", "600550", "600558", "600561", "600565", "600566", "600563", "600567", "600570", "600568", "600569", "600572", "600571", "600115", "600152", "600129", "600151", "600118", "600153", "600595", "600596", "600599", "600597", "600600", "600601", "600598", "600603", "600602", "600575", "600580", "600577", "600584", "600573", "600583", "600586", "600587", "600588", "600592", "600593", "600604", "600576", "600579", "600578", "600585", "600594", "600581", "600605", "600609", "600589", "600612", "600582", "600614", "600590", "600613", "600615", "600620", "600616", "600611", "600606", "600619", "600621", "600622", "600617", "600618", "600629", "600623", "600630", "600628", "600626", "600633", "600635", "600639", "600636", "600624", "600637", "600638", "600640", "600642", "600643", "600644", "600641", "600647", "600649", "600645", "600652", "600653", "600648", "600651", "600657", "600660", "600650", "600661", "600658", "600655", "600662", "600666", "600663", "600664", "600665", "600667", "600673", "600668", "600674", "600677", "600675", "600678", "600676", "600681", "600671", "600679", "600682", "600683", "600686", "600688", "600684", "600689", "600685", "600693", "600687", "600692", "600690", "600691", "600698", "600694", "600695", "600697", "600703", "600705", "600699", "600702", "600707", "600706", "600710", "600712", "600704", "600711", "600714", "600708", "600713", "600715", "600716", "600717", "600718", "600719", "600721", "600720", "600722", "600723", "600726", "600727", "600729", "600728", "600730", "600724", "600731", "600734", "600735", "600736", "600737", "600738", "600739", "600743", "600740", "600744", "600741", "600745", "600746", "600742", "600750", "600754", "600751", "600748", "600753", "600755", "600758", "600757", "600759", "600761", "600760", "600763", "600765", "600766", "600756", "600768", "600764", "600771", "600769", "600770", "600773", "600774", "600776", "600775", "600780", "600782", "600779", "600781", "600783", "600784", "600787", "600795", "600796", "600777", "600785", "600800", "600792", "600797", "600791", "600801", "600789", "600790", "600793", "600802", "600803", "600798", "600794", "600809", "600812", "600815", "600804", "600814", "600805", "600808", "600811", "600810", "600819", "600820", "600816", "600821", "600818", "600822", "600823", "600830", "600824", "600831", "600825", "600826", "600827", "600828", "600836", "600834", "600835", "600829", "600833", "600838", "600837", "600841", "600844", "600847", "600843", "600845", "600850", "600846", "600851", "600855", "600854", "600839", "600848", "600853", "600856", "600857", "600859", "600858", "600862", "600860", "600861", "600865", "600867", "600866", "600869", "600863", "600868", "600872", "600873", "600864", "600874", "600875", "600876", "600879", "600880", "600883", "600884", "600886", "600887", "600888", "600889", "600885", "600890", "600882", "600881", "600891", "600892", "600894", "600895", "600898", "600897", "600908", "600900", "600893", "600917", "600909", "600901", "600919", "600903", "600926", "600933", "600929", "600959", "600939", "600962", "600961", "600958", "600960", "600966", "600963", "600936", "600965", "600969", "600971", "600973", "600975", "600967", "600970", "600976", "600979", "600978", "600977", "600981", "600982", "600980", "600984", "600985", "600983", "600987", "600992", "600990", "600995", "600988", "600986", "600993", "600996", "600997", "600999", "601001", "601005", "600998", "601000", "601002", "601006", "601007", "601009", "601008", "601003", "601010", "601015", "601011", "601018", "601028", "601012", "601019", "601016", "601038", "601069", "601058", "601068", "601099", "601101", "601020", "601100", "601086", "601021", "601066", "601106", "601098", "601107", "601088", "601111", "601108", "601118", "601137", "601138", "601117", "601127", "601116", "601139", "601155", "601163", "601158", "601169", "601113", "601126", "601166", "601179", "601177", "601128", "601168", "601208", "601186", "601200", "601188", "601211", "601212", "601198", "601199", "601222", "601226", "601216", "601218", "601225", "601229", "601228", "601238", "601231", "601233", "601288", "601311", "601258", "601318", "601330", "601328", "601313", "601333", "601326", "601336", "601339", "601366", "601360", "601369", "601375", "601368", "601377", "601388", "601390", "601500", "601518", "601555", "601566", "601515", "601398", "601579", "601519", "601588", "601567", "601595", "601606", "601600", "601599", "601607", "601616", "601611", "601608", "601601", "601618", "601666", "601619", "601628", "601633", "601669", "601677", "601668", "601688", "601689", "601699", "601678", "601636", "601718", "601700", "601717", "601766", "601727", "601777", "601789", "601800", "601808", "601811", "601788", "601799", "601838", "601801", "601818", "601857", "601828", "601858", "601869", "601877", "601866", "601886", "601881", "601878", "601882", "601872", "601880", "601890", "601899", "601888", "601898", "601900", "601901", "601908", "601928", "601918", "601929", "601949", "601919", "601939", "601933", "601965", "601958", "601952", "601969", "601985", "601968", "601989", "601990", "601991", "601966", "601988", "601992", "601998", "601996", "601999", "603000", "601997", "603002", "603001", "603003", "603007", "603010", "603008", "603006", "603005", "603009", "603011", "603012", "603015", "603013", "603017", "603019", "603018", "603020", "603016", "603022", "603023", "603025", "603027", "603021", "603030", "603028", "603026", "603029", "603031", "603032", "603033", "603036", "603035", "603039", "603037", "603040", "603043", "603042", "603045", "603050", "603055", "603038", "603041", "603056", "603058", "603059", "603063", "603060", "603067", "603069", "603076", "603078", "603077", "603079", "603066", "603081", "603080", "603083", "603085", "603086", "603089", "603088", "603090", "603096", "603098", "603101", "603100", "603099", "603103", "603105", "603108", "603106", "603110", "603116", "603111", "603113", "603118", "603117", "603123", "603126", "603128", "603127", "603131", "603133", "603136", "603138", "603139", "603129", "603156", "603158", "603159", "603161", "603160", "603165", "603157", "603166", "603168", "603169", "603167", "603178", "603181", "603177", "603179", "603183", "603186", "603189", "603196", "603180", "603192", "603197", "603198", "603200", "603199", "603203", "603222", "603214", "603218", "603225", "603226", "603227", "603223", "603229", "603208", "603238", "603228", "603232", "603233", "603258", "603239", "603260", "603259", "603266", "603268", "603269", "603319", "603320", "603321", "603333", "603335", "603322", "603326", "603323", "603328", "603336", "603329", "603330", "603277", "603331", "603278", "603283", "603337", "603289", "603301", "603286", "603300", "603338", "603298", "603288", "603303", "603305", "603299", "603306", "603311", "603308", "603318", "603309", "603313", "603315", "603345", "603316", "603348", "603339", "603357", "603360", "603356", "603355", "603368", "603365", "603366", "603377", "603367", "603383", "603378", "603358", "603359", "603363", "603387", "603369", "603386", "603380", "603385", "603388", "603389", "603396", "603398", "603399", "603393", "603456", "603416", "603458", "603421", "603444", "603429", "603486", "603496", "603466", "603488", "603500", "603505", "603501", "603499", "603477", "603507", "603506", "603508", "603517", "603515", "603519", "603528", "603518", "603527", "603516", "603520", "603533", "603555", "603536", "603535", "603538", "603556", "603566", "603567", "603557", "603577", "603558", "603559", "603568", "603579", "603578", "603569", "603585", "603580", "603586", "603587", "603588", "603596", "603598", "603589", "603599", "603590", "603601", "603595", "603603", "603600", "603606", "603602", "603605", "603612", "603609", "603607", "603608", "603616", "603611", "603617", "603615", "603626", "603619", "603618", "603630", "603638", "603637", "603636", "603633", "603628", "603648", "603639", "603655", "603658", "603650", "603659", "603656", "603660", "603657", "603665", "603666", "603667", "603661", "603668", "603669", "603677", "603663", "603678", "603676", "603685", "603679", "603680", "603683", "603686", "603688", "603690", "603693", "603698", "603703", "603699", "603706", "603689", "603696", "603709", "603707", "603701", "603711", "603712", "603708", "603716", "603718", "603717", "603721", "603728", "603726", "603727", "603713", "603733", "603722", "603725", "603730", "603737", "603738", "603729", "603767", "603757", "603768", "603773", "603776", "603758", "603766", "603819", "603828", "603822", "603825", "603838", "603826", "603823", "603829", "603839", "603843", "603833", "603848", "603860", "603866", "603868", "603876", "603869", "603855", "603856", "603858", "603859", "603871", "603861", "603878", "603880", "603879", "603881", "603882", "603883", "603877", "603887", "603889", "603890", "603888", "603898", "603896", "603897", "603895", "603903", "603885", "603886", "603899", "603900", "603906", "603901", "603778", "603777", "603779", "603797", "603787", "603798", "603788", "603799", "603789", "603800", "603801", "603806", "603803", "603809", "603811", "603816", "603813", "603817", "603808", "603818", "603908", "603912", "603909", "603916", "603917", "603918", "603919", "603922", "603926", "603929", "603933", "603920", "603936", "603937", "603928", "603938", "603939", "603959", "603955", "603960", "603958", "603966", "603969", "603968", "603976", "603963", "603977", "603970", "603978", "603980", "603979", "603985", "603988", "603987", "603986", "603989", "603990", "603991", "603996", "603993", "603997", "603998", "603999", "600157", "600158", "600165", "600171", "600175", "600180", "600183"]
```

###### 爬虫代码
```
# -*- coding:utf-8 -*-
# filename: sse_com_cn_company_info_selenium.py

import json
import random
import pymongo
# import asyncio

from prettytable import PrettyTable
from selenium import webdriver
from selenium.webdriver.support import expected_conditions as EC
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait

header = {}
header['referer'] = 'http://listxbrl.sse.com.cn/companyInfo/toCompanyInfo.do?stock_id=600000&report_period_id=5000'

class CompanyInfoFetcher():
    def __init__(self,timeout=20,options=None):
        self.timeout = timeout
        self.options = options
        self.browser = webdriver.Chrome(options=self.options)
        # 设置超时对象
        self.wait = WebDriverWait(self.browser, self.timeout)
        header['user-agent'] = self.get_user_agent()
        # 数据库初始化
        client = pymongo.MongoClient('localhost')
        db = client['shangjiaosuo_sse_com_cn']
        self.collection = db['company_years_meta_info']

    def access_pages(self):
        print('45-45')
        stock_codes = self.get_stock_codes()
        for stock_code in stock_codes:
            self.browser.get('http://listxbrl.sse.com.cn/companyInfo/toCompanyInfo.do?stock_id={0:s}&report_period_id=5000'.format(stock_code))
            # 等待页面表格数据元素加载出来
            self.wait.until(
                EC.presence_of_element_located((By.XPATH, '/html[1]/body[1]/div[5]/div[1]/div[1]/div[1]/div[1]/div[2]/div[2]/table[1]/tbody[1]/tr[1]/td[2]/div[1]/div[1]')))
            self.parse_page(stock_code)

    def parse_page(self, stock_code):
        '''
        在浏览器中执行以下js代码可以获得 metas 数组
        [].slice.call(document.querySelectorAll('body.easyui-layout.layout:nth-child(2) div.panel.layout-panel.layout-panel-center:nth-child(8) div.panel-body.panel-body-noheader.panel-body-noborder.layout-body.panel-noscroll div.panel.datagrid.propertygrid:nth-child(1) div.datagrid-wrap.panel-body.panel-body-noheader div.datagrid-view div.datagrid-view2:nth-child(3) div.datagrid-body table.datagrid-btable tbody:nth-child(1) tr.datagrid-row td:nth-child(1) > div.datagrid-cell.datagrid-cell-c6-name')).map(item => item.innerText)
        -----------------------------------------
        # 在浏览器中执行以下js代码可以获得 years 数组
        [].slice.call(document.querySelectorAll(
            'body.easyui-layout.layout:nth-child(2) div.panel.layout-panel.layout-panel-center:nth-child(8) div.panel-body.panel-body-noheader.panel-body-noborder.layout-body.panel-noscroll div.panel.datagrid.propertygrid:nth-child(1) div.datagrid-wrap.panel-body.panel-body-noheader div.datagrid-view div.datagrid-view2:nth-child(3) div.datagrid-header div.datagrid-header-inner table.datagrid-htable tbody:nth-child(1) tr.datagrid-header-row td div.datagrid-cell > span:nth-child(1)')).map(
            item => item.innerText).slice(1)
        '''
        metas = ["公司法定中文名称", "公司法定代表人", "公司注册地址", "公司办公地址邮政编码", "公司国际互联网网址", "公司董事会秘书姓名", "公司董事会秘书电话", "公司董事会秘书电子信箱", "报告期末股东总数", "每10股送红股数", "每10股派息数（含税）", "每10股转增数", "本期营业收入(元)", "本期营业利润(元)", "利润总额(元)", "归属于上市公司股东的净利润(元)", "归属于上市公司股东的扣除非经常性损益的净利润(元)", "经营活动产生的现金流量净额(元)", "总资产(元)", "所有者权益（或股东权益）(元)", "基本每股收益(元/股)", "稀释每股收益(元/股)", "扣除非经常性损益后的基本每股收益(元/股)", "全面摊薄净资产收益率（%）", "加权平均净资产收益率（%）", "扣除非经常性损益后全面摊薄净资产收益率（%）", "扣除非经常性损益后的加权平均净资产收益率（%）", "每股经营活动产生的现金流量净额(元/股)", "归属于上市公司股东的每股净资产（元/股）"]
        years = ["2017", "2016", "2015", "2014", "2013"]

        table = PrettyTable(['{0:s} 指标/年份'.format(stock_code)] + years)

        company = {}
        for i, meta in enumerate(metas):
            row_num = int(i) + 1
            for j, year in enumerate(years):
                col_num = int(j) + 1
                # 全部提取代码都在这一行了
                value = self.browser.find_element_by_xpath('/html[1]/body[1]/div[5]/div[1]/div[1]/div[1]/div[1]/div[2]/div[2]/table[1]/tbody[1]/tr[{0:d}]/td[{1:d}]/div[1]'.format(row_num, col_num)).text
                if not meta in company:
                    company[meta] = {}
                company[meta][year] = value
            table.add_row([meta] + list(company[meta].values()))
        self.collection.insert_one(company)
        print(table)
        print('{0:<50}{1:10}{0:>50}'.format('*' * 30, '分割线'))

    def get_stock_codes(self):
        try:
            with open('shangjiaoshuo_stock_codes.list.json', 'r') as f:
                stock_codes = json.loads(f.read())
                # print(self.stock_codes)
            f.close()
            return  stock_codes
        except Exception:
            raise(Exception)

    def get_user_agent(self):
        return random.choice([
            "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_9_3) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/35.0.1916.47 Safari/537.36",
            "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/51.0.2704.103 Safari/537.36",
            "Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/57.0.2987.133 Safari/537.36",
            "Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.1; SV1; AcooBrowser; .NET CLR 1.1.4322; .NET CLR 2.0.50727)",
            "Mozilla/4.0 (compatible; MSIE 7.0; Windows NT 6.0; Acoo Browser; SLCC1; .NET CLR 2.0.50727; Media Center PC 5.0; .NET CLR 3.0.04506)",
            "Mozilla/4.0 (compatible; MSIE 7.0; AOL 9.5; AOLBuild 4337.35; Windows NT 5.1; .NET CLR 1.1.4322; .NET CLR 2.0.50727)",
            "Mozilla/5.0 (Windows; U; MSIE 9.0; Windows NT 9.0; en-US)",
            "Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; Win64; x64; Trident/5.0; .NET CLR 3.5.30729; .NET CLR 3.0.30729; .NET CLR 2.0.50727; Media Center PC 6.0)",
            "Mozilla/5.0 (compatible; MSIE 8.0; Windows NT 6.0; Trident/4.0; WOW64; Trident/4.0; SLCC2; .NET CLR 2.0.50727; .NET CLR 3.5.30729; .NET CLR 3.0.30729; .NET CLR 1.0.3705; .NET CLR 1.1.4322)",
            "Mozilla/4.0 (compatible; MSIE 7.0b; Windows NT 5.2; .NET CLR 1.1.4322; .NET CLR 2.0.50727; InfoPath.2; .NET CLR 3.0.04506.30)",
            "Mozilla/5.0 (Windows; U; Windows NT 5.1; zh-CN) AppleWebKit/523.15 (KHTML, like Gecko, Safari/419.3) Arora/0.3 (Change: 287 c9dfb30)",
            "Mozilla/5.0 (X11; U; Linux; en-US) AppleWebKit/527+ (KHTML, like Gecko, Safari/419.3) Arora/0.6",
            "Mozilla/5.0 (Windows; U; Windows NT 5.1; en-US; rv:1.8.1.2pre) Gecko/20070215 K-Ninja/2.1.1",
            "Mozilla/5.0 (Windows; U; Windows NT 5.1; zh-CN; rv:1.9) Gecko/20080705 Firefox/3.0 Kapiko/3.0",
            "Mozilla/5.0 (X11; Linux i686; U;) Gecko/20070322 Kazehakase/0.4.5",
            "Mozilla/5.0 (X11; U; Linux i686; en-US; rv:1.9.0.8) Gecko Fedora/1.9.0.8-1.fc10 Kazehakase/0.5.6",
            "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/535.11 (KHTML, like Gecko) Chrome/17.0.963.56 Safari/535.11",
            "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_7_3) AppleWebKit/535.20 (KHTML, like Gecko) Chrome/19.0.1036.7 Safari/535.20",
            "Opera/9.80 (Macintosh; Intel Mac OS X 10.6.8; U; fr) Presto/2.9.168 Version/11.52",
            "Mozilla/5.0 (Windows; U; Windows NT 5.2) Gecko/2008070208 Firefox/3.0.1",
            "Mozilla/5.0 (Windows; U; Windows NT 5.1) Gecko/20070309 Firefox/2.0.0.3",
            "Mozilla/5.0 (Windows; U; Windows NT 5.1) Gecko/20070803 Firefox/1.5.0.12",
            "Opera/9.27 (Windows NT 5.2; U; zh-cn)",
            "Mozilla/5.0 (Windows; U; Windows NT 5.2) AppleWebKit/525.13 (KHTML, like Gecko) Version/3.1 Safari/525.13",
            "Mozilla/5.0 (iPhone; U; CPU like Mac OS X) AppleWebKit/420.1 (KHTML, like Gecko) Version/3.0 Mobile/4A93 ",
            "Mozilla/5.0 (Windows; U; Windows NT 5.2) AppleWebKit/525.13 (KHTML, like Gecko) Chrome/0.2.149.27 ",
            "Mozilla/5.0 (Linux; U; Android 3.2; ja-jp; F-01D Build/F0001) AppleWebKit/534.13 (KHTML, like Gecko) Version/4.0 Safari/534.13 ",
            "Mozilla/5.0 (iPhone; U; CPU iPhone OS 4_1 like Mac OS X; ja-jp) AppleWebKit/532.9 (KHTML, like Gecko) Version/4.0.5 Mobile/8B117 Safari/6531.22.7",
            "Mozilla/5.0 (iPhone; U; CPU iPhone OS 4_2_1 like Mac OS X; da-dk) AppleWebKit/533.17.9 (KHTML, like Gecko) Version/5.0.2 Mobile/8C148 Safari/6533.18.5 ",
            "Mozilla/5.0 (Macintosh; U; Intel Mac OS X 10_5_6; en-US) AppleWebKit/530.9 (KHTML, like Gecko) Chrome/ Safari/530.9 ",
            "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_7_0) AppleWebKit/535.11 (KHTML, like Gecko) Chrome/17.0.963.56 Safari/535.11",
            "Mozilla/4.0 (compatible; MSIE 7.0; Windows NT 5.1; 360SE)",
            "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/535.11 (KHTML, like Gecko) Ubuntu/11.10 Chromium/27.0.1453.93 Chrome/27.0.1453.93 Safari/537.36",
            "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_7_2) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/27.0.1453.93 Safari/537.36",
            "Mozilla/5.0 (Windows NT 6.2; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/27.0.1453.94 Safari/537.36",
            "Mozilla/5.0 (Linux; Android 5.1.1; Nexus 6 Build/LYZ28E) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/63.0.3239.84 Mobile Safari/537.36"
        ])

if __name__ == '__main__':
    fetcher = CompanyInfoFetcher()
    fetcher.access_pages()

```

## 执行爬取
```
python filename: sse_com_cn_company_info_selenium.py
```