### ~2018/05/08
1、尝试将Griffli-Lim嵌入到voice_convertion的convert中，提高语音合成的效率>>参数不一致，调参过程中;
2、调整net2模型训练参数，降低loss值，优化模型>>优化后，loss值从1.2下降到0.8；
3、减少喂给net2的样本数量，训练小样本模型，对比模型效果>>模型效果略有下降，不是很明显；
4、替换中文数据样本库，训练net1模型，评估模型效果，尝试优化>>loss曲线振荡，下降不明显；
5、采用英文数据集的net1参数，Fine_tune中文数据集的net1模型训练>>模型不能匹配;
5、将英文音节库补齐中文音节，先用英文数据集训练，再用中文数据集Fine_tune>>模型训练过程中.

5/8
基于基频和共振峰等声音特征改变声音的音色、音调研究：
1、demo1:通过重采样调整声音频率达到一定的变音效果，已编写代码实现功能;
2、demo2:基于demo1基础，结合语音线性预测（LPC）、合成滤波器等算法进一步提升功能，已编写代码跑通，但效果不理想需进一步优化。

5/9
参考文献：https://wenku.baidu.com/view/ce2cb01748d7c1c708a145bc.html?from=search


2018-5-8 周二 蒋成林--工作小结
把F类识别和性别识别和改成离线版，完善打包脚本与选项代码，远程部署在凡普金科服务器上；
性别识别因环境问题出错，尚未部署完成。
蒋成林_工作状态

2018年5月:
意图识别：数据准备阶段；

意图打标系统：开发完善阶段；

脏话识别：接口开发阶段；模型评估阶段；

使用的代码库：
t@192.168.1.68:jiang/labeler.git      意图打标系统
git@192.168.1.68:jiang/intent_corpus.git        意图语料库
git@192.168.1.68:jiang/Rasa_NLU_Chi.git     Rasa 框架
git@192.168.1.68:jiang/keyword_extract.git      关键词提取



### 2018/5/23
**关键词提取系统：**

	1、任务总体流程梳理和任务细化：
		  >>>运行，从数据库抓取数据，抓取条件设置为某个时间段
              >>>进行分词，计算词频，通过设置阈值进行筛选
              >>>将筛选结果存入数据库暂存列表，并在页面进行展示
              >>>人工进行筛选，将最终结果存入数据表里
              >>>可通过词频排序构建新的模板
	2、数据库结构 （两个表）
               id; sentense; datatime; username
		id; word ; TF; TF-IDF; username; datetime
	3、页面设计
		时间段的选择框、运行按钮（进行分词并筛选）;
		展示按钮，将自动筛选的关键结果全部展示，人工进行点击筛选并保存;
***
### 2018/5/25
**Dophin**

	启动命令：
	```python3 assistant/client.py -a 127.0.0.1 -p 10000 -c unittest```

	给文件修改权限
	```sudo chown -R adalla(username) sellbot/(filename)```
***

### 2018/5/29
	NLU：在忙 enter 没有了
	TTS打标：35205-35940 clear or noise;37994-38243 voice or noise;
***

### 2018/5/30
一、向mysql数据库中写入中文，报错问题解决解决方法：

		1.自动新建的这张表的character set 没有设置成utf8.
		2.服务器向数据库中传输的数据格式没有设置成uft8。因此，先进入数据库：
		  (1)在cmd命令窗口输入 mysql -hlocalhost -uroot -p回车 进入mysql数据库;
		  (2)查看表中的各字段的属性： show full columns from table_name;
		  如果看到每个字段都是lati， 则执行以下语句：
	  	alter table table_name convert to character set utf8 collate     utf8_general_ci;
		  (3)同时检查一些变量collation_connection、collation_database、collation_server是否都是utf8_general_ci
		  检查的语句为：show variable like 'collation%';
		  修改的语句为：set collation_connection=utf8_general_ci
		  (4)同时检查一些变量character的相关变量是否为utf8_general_ci
		  检查的语句为：show variable like 'character%';
	   	修改语句如：set character_set_database=utf8;

二、UBUNTU安装FFMPEG
```
		sudo add-apt-repository ppa:djcj/hybrid
		sudo apt-get update
		sudo apt-get install ffmpeg
```
三、话术系统

***
### 2018/5/31
一、话术音频格式转换
```python
#test
def download_wavs(self):
		url = "http://global.res.btows.com/upload/voice/201805/20180530141006730.wav"
		if url:
				cfg_name = self.temp_json["key_str"]
				if cfg_name.endswith('_en'):
						cfg_name = cfg_name[:-3]
				wav_path_orig = os.path.join(self.tem_path, '%s/wav/%s_rec_orig' % (cfg_name, cfg_name))
				print(self.tem_path)
				wav_path_orig = os.path.realpath(wav_path_orig)
				wav_path = os.path.join(self.tem_path, '%s/wav/%s_rec' % (cfg_name, cfg_name))
				wav_path = os.path.realpath(wav_path)
				mkdir_p(wav_path_orig) #原始音频下载保存路径
				mkdir_p(wav_path) #转格式音频路径

				fn_orig = os.path.join(wav_path_orig, '1.wav')
				download(url, fn_orig)
				fn = os.path.join(wav_path,  '1.wav')
				os.system('ffmpeg -i %s -f wav -ar 16000 -ac 1 %s' % (fn_orig, fn))
```
```python
# deve
#音频保存路径：/home/xupeng/workspace/visual_templates/dmf_52974/wav/dmf_52974_rec
def download_wavs(self):
bot_sentences = self.temp_json["bot_sentences"]
for item in bot_sentences:
    url = item['file']
    if url:
        cfg_name = self.temp_json["key_str"]
        if cfg_name.endswith('_en'):
            cfg_name = cfg_name[:-3]
        wav_path_orig = os.path.join('/home/xupeng/workspace/visual_templates/', '%s/wav/%s_rec_orig' % (cfg_name, cfg_name))
        wav_path_orig = os.path.realpath(wav_path_orig)
        wav_path = os.path.join('/home/xupeng/workspace/visual_templates/', '%s/wav/%s_rec' % (cfg_name, cfg_name))
        wav_path = os.path.realpath(wav_path)
        mkdir_p(wav_path_orig) #原始音频下载保存路径
        mkdir_p(wav_path) #转格式音频路径

        fn_orig = os.path.join(wav_path_orig, item['num'] + '.wav')
        download(url, fn_orig)
        fn = os.path.join(wav_path, item['num'] + '.wav')
        os.system('ffmpeg -i %s -f wav -ar 8000 -ac 1 %s' % (fn_orig, fn))
```
二、获取根目录
```python
home = os.path.expanduser('~')
```
三、获取本地ip
```python
def get_ip():
    s = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
    try:
        s.connect(('8.8.8.8', 80))
        ip = s.getsockname()[0]
    finally:
        s.close()
    return ip
```
四、[SOX and Lame](https://blog.csdn.net/Liao_Jingyi/article/details/48480913)

**[SoX](/home/adalla/jiangda/LOG/链接/play_sox.py)-声音转换、音频处理中的瑞士军刀**
***
### 2018/6/1
一、sellbot部署/测试
* 三个平级目录：cfg_repos;sellbot3(single);sellbot_scripts(dev).
* 将模板放入cfg_repo
* sellbot3终端运行：python3 app.py -c 模板名;python3 client.py -c 模板名; . m; 4;
* 154密码：og.5XcCeumfk
* ssh root@192.168.1.171  toolwiz.com
* cd /home/sellbot
* cd var/www/html
* cp test.php test2.php >>> php xxx

	**代运营、财税、保险：**
	保险 zgrscxcx
	checking select.json for keywords conflict ...
	select.json 一般问题 and 在忙 有相似关键词 ['再说 vs. 再说', '再说 vs. 再跟你联系->再说', '在忙 vs. 出差->在忙', '在忙 vs. 现在有点事情->在忙', '晚点 vs. 晚点', '晚点再联系 vs. 晚点再联系', '没时间不方便 vs. 最近没空->没时间不方便', '没时间不方便 vs. 没时间不方便']
	select.json 一般问题 and 拒绝 有相似关键词 ['买了 vs. 买了', '打错 vs. 打错']

二、批量域名导入模板
```python
def json2db_batch(cfg_name_or_path, new_name=None, prefix='', DOMAIN_NAMES=None,
            update=False, add_or_update=False, upload_wav=True, base=True):
    try:
        data = run_domain_2_db(cfg_name_or_path=cfg_name_or_path,
                               new_name=new_name, prefix=prefix, upload_wav=upload_wav)
        for domain_name in DOMAIN_NAMES:
            print('正在导入模板 %s 到 %s 服务器' % (cfg_name_or_path, domain_name))
            add_cnt = 0
            update_cnt = 0
            fails = []
            if add_or_update:
                result = update_after_add_cfg(data, domain_name)
            else:
                result = add_or_update_cfg(data, domain_name, update=update, base=base)

            rt = 0  # db fail
            if result == 'success':
                rt = 1
            elif result == 'update success':
                rt = 2

            if rt == 1:
                add_cnt += 1
            elif rt == 2:
                update_cnt += 1
            else:
                fails.append(cfg_name_or_path)

            if rt not in [1, 2]:
                s = input('result %s, 是否继续？y/n' % str(rt))
                if len(s) > 0 and s.lower() != 'y':
                    break
            print('add: %d update: %d fail: %d' % (add_cnt, update_cnt, len(fails)))
            print('fail:', fails)
            # return rt
    except Exception as e:
        # traceback.print_exc()
        exc_info = traceback.format_exc()
        print('json2db error exc_info:', exc_info)
        print('json2db error:', str(e))
        return -1
```
```python
def json2db(cfg_name_or_path, new_name=None, prefix='', domain_name='test.btows.com',
            update=False, add_or_update=False, upload_wav=True, base=True):
    print('正在导入模板 %s 到 %s 服务器' % (cfg_name_or_path, domain_name))
    try:
        data = run_domain_2_db(cfg_name_or_path=cfg_name_or_path,
                               new_name=new_name, prefix=prefix, upload_wav=upload_wav)

        if add_or_update:
            result = update_after_add_cfg(data, domain_name)
        else:
            result = add_or_update_cfg(data, domain_name, update=update, base=base)

        rt = 0  # db fail
        if result == 'success':
            rt = 1
        elif result == 'update success':
            rt = 2
        return rt
    except Exception as e:
        # traceback.print_exc()
        exc_info = traceback.format_exc()
        print('json2db error exc_info:', exc_info)
        print('json2db error:', str(e))
        return -1
```
```python
def batch_import():
    path_list = ["pos机/zyposj", "保险/zgrscxcx", "财税/bjhs", "代运营/xzdyy", "法律服务/hbsx", "房产/cbhkqc", "股票/bszbsgp",
                 "回访/tkrs", "活动通知/whcbh", "基金/xhcflc2", "机器人/gyqdzs", "酒水/zqqmtj", "美容/szcct", "信用卡/xyyhdxfy",
                 "装修/jszs", "资质办理/szslbgx", "贷款/dkgxb2"]

    used_trades = '贷款、装修、信用卡、股票、房产、法律服务、代运营、财税、保险'.split('、')

    path_list = ["财税/bjhs"]

    for path in path_list:
        trade, cfg_name = path.split('/')
        if trade not in used_trades:
            continue

        json2db_batch('./trade_cfgs/' + path + '/cfgs/%s' % cfg_name, prefix='trade_',
                         add_or_update=True, upload_wav=True, DOMAIN_NAMES=DOMAIN_NAMES)
```
***
### 2018/6/4
一、[TF—IDF](https://www.cnblogs.com/chenbjin/p/3851165.html)

二、[Jieba](https://www.cnblogs.com/eastmount/p/5055906.html)
   [工具的使用](https://blog.csdn.net/qq_27231343/article/details/51898940)

三、[Git命令](/home/adalla/jiangda/LOG/链接/420821552.jpg)
***
### 2018/6/5
一、keywords_extract

* [数据抓取](/home/adalla/jiangda/LOG/链接/keywords_data_man.py)
***
### 2018/6/6
一、合并抓取数据的时间区间，给前端进行显示对应行业已抓取数据的时间区间
二、数据库的导入和导出
```
１．导出数据库
命令格式：mysqldump -u用户名 -p 数据库名 > 数据库名.sql
例如：mysqldump -uroot -p abc > abc.sql　　（导出数据库abc到abc.sql文件）
２．导入数据库
命令格式：mysql -u用户名 -p 数据库名 < 数据库名.sql
例如：mysql -urootf -p abc < abc.sql　　　　（导出数据库abc到abc.sql文件）
3. 导出单个数据表结构和数据
mysqldump -h localhost -uroot -p123456  database table > dump.sql
4. 导出整个数据库结构（不包含数据）
mysqldump -h localhost -uroot -p123456  -d database > dump.sql
5. 导出单个数据表结构（不包含数据）
mysqldump -h localhost -uroot -p123456  -d database table > dump.sql
```
***
### 2018/6/7
一、mysql删除表中数据
```
设置mysql安全模式，解除
SET SQL_SAFE_UPDATES = 0;
程度从强到弱
1、drop  table tb
      drop将表格直接删除，没有办法找回
2、truncate (table) tb
      删除表中的所有数据，不能与where一起使用
3、delete from tb (where)
      删除表中的数据(可制定某一行)
```
二、[关键词系统早期dataman代码草稿](/home/adalla/jiangda/LOG/链接/dataman_old_keywords.py)
***
### 2018/06/08
一、电话音嘟声识别
* 傅里叶变换到频域
* 互相关系数：numpy.corrcoef (皮尔逊相关系数)

二、[Deep Learning Papers

 **[1]**  [Reading](Roadmaphttps://github.com/floodsung/Deep-Learning-Papers-Reading-Roadmap):star::star::star::star::star:
**[2]** [重磅干货](https://blog.csdn.net/zhongwen7710/article/details/45331915):star::star::star::star::star:

**[3]** Do _update_unused & _compare_with_used
```python
def _update_unused(self,industry, start_time="2018-05-01", end_time="2018-05-02"):
		unused = self._compare_with_used(industry, start_time=start_time, end_time=end_time)
		self.trade_dict[industry] = unused
		self.trade_keywords_dict['unused'] = self.trade_dict

def _compare_with_used(self, industry, start_time="2018-05-01", end_time="2018-05-02"):
    keywords_list = []
    d_m = DataMan()
    r = d_m.algorithm(industry=industry, start_time=start_time, end_time=end_time)
    for n in r:
        keywords_list.append(n[0])
    print("past:", keywords_list)
    if industry in self.trade_keywords_dict["used"]:
        for w in keywords_list:
            if w in self.trade_keywords_dict["used"][industry]:
                keywords_list.remove(w)
        print("now:", keywords_list)
    unused = keywords_list
    return unused
```
