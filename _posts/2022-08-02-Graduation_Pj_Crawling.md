---
layout: post
title: Graduation_Pj
subtitle: Crawling
categories: Graduation_Pj(Crawling)
tags: Crawling Graduation_Pj
---

### Graduation_Pj
 
4학년 1학기 졸업 팀 프로젝트를 수행하는 중 이를 기록하고 싶어 중간 단계 이지만 기록을 시작하였습니다

### 프로젝트 목표

학우들의 학업을 돕기 위해 새로운 과제, 제출 기한이 얼마 남지 않은 과제에 대한 PYSH알림을 주는 어플을 만들어 과제를 까먹는 불상사를 막기위해 프로젝트를 계획하고 실시하였습니다.

### 내가 할 일

Crawling을 통해 학교 홈페이지에 접근하여 과제에 대한 필요한 데이터를 저장하여 서버에 넘겨 주는 것을 수행하였습니다.

---

### 첫번 째 Crawling 코드

```Python
# pip install webdriver-manager # 크롬드라이버 자동설치
# pip install selenium
# pip install bs4
# pip install lxml

from concurrent.futures import process
from urllib import response
from xml.dom.minidom import Element
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
from selenium.common.exceptions import NoSuchElementException # 예외지정
from selenium.webdriver.chrome.service import Service
from webdriver_manager.chrome import ChromeDriverManager
from bs4 import BeautifulSoup as bs
from selenium.webdriver.common.desired_capabilities import DesiredCapabilities # 추가

import time
import bs4
import json
import re

chrome_options = webdriver.ChromeOptions()

# 브라우저 창 없이 실행
chrome_options.add_argument("--headless")
chrome_options.add_argument("--start-maximized")
chrome_options.add_argument("--window-size=1920,1080")
chrome_options.add_argument("--disable-gpu")  # 추가
chrome_options.add_argument("--disable-infobars") # 추가
chrome_options.add_argument("--disable-extensions") # 추가
chrome_options.add_argument("--disable-popup-blocking") # 추가

# 속도 향상을 위한 옵션 해제 추가
prefs = {'profile.default_content_setting_values': {'cookies' : 2, 'images': 2, 'plugins' : 2, 'popups': 2, 'geolocation': 2, 'notifications' : 2, 'auto_select_certificate': 2, 'fullscreen' : 2, 'mouselock' : 2, 'mixed_script': 2, 'media_stream' : 2, 'media_stream_mic' : 2, 'media_stream_camera': 2, 'protocol_handlers' : 2, 'ppapi_broker' : 2, 'automatic_downloads': 2, 'midi_sysex' : 2, 'push_messaging' : 2, 'ssl_cert_decisions': 2, 'metro_switch_to_desktop' : 2, 'protected_media_identifier': 2, 'app_banner': 2, 'site_engagement' : 2, 'durable_storage' : 2}}   
chrome_options.add_experimental_option('prefs', prefs) # 추가

caps = DesiredCapabilities().CHROME  # 추가
caps["pageLoadStrategy"] = "none"   # 추가

# Chromedriver 경로 설정
browser = webdriver.Chrome(service = Service(ChromeDriverManager().install()), options = chrome_options)

# url 이동
browser.get("https://portal.gwnu.ac.kr/user/login.face?ssoReturn=https://lms.gwnu.ac.kr")

# 로그인 정보
userid = "20211954"
password = "971126"

# id, pw 입력
idBox = browser.find_element(By.ID, "userId")
pwBox = browser.find_element(By.ID, "password")
browser.execute_script("arguments[0].value=arguments[1]", idBox, userid) # 추가
browser.execute_script("arguments[0].value=arguments[1]", pwBox, password) # 추가

# 로그인 버튼 클릭
WebDriverWait(browser, 5).until(EC.element_to_be_clickable((By.XPATH, "/html/body/div/div/div[1]/div/div/div[1]/a"))).click()

# 현재 수강 과목 리스트로 저장 (최대 8개)
subject_list = ['//*[@id="mCSB_1_container"]/li[1]/a/span[1]',
                '//*[@id="mCSB_1_container"]/li[2]/a/span[1]',
                '//*[@id="mCSB_1_container"]/li[3]/a/span[1]',
                '//*[@id="mCSB_1_container"]/li[4]/a/span[1]',
                '//*[@id="mCSB_1_container"]/li[5]/a/span[1]',
                '//*[@id="mCSB_1_container"]/li[6]/a/span[1]',
                '//*[@id="mCSB_1_container"]/li[7]/a/span[1]',
                '//*[@id="mCSB_1_container"]/li[8]/a/span[1]']


# 과제에 대한 리스트 선언
title_result = []
d_day_start_result = []
d_day_end_result = []
content_result = []
d_end_result = []
course_result = []
clear_result = []
progress_result = []

# json 리스트 선언
dict_key = []
temp_dict = {}

# 과제 정보 가져오기
for i in subject_list :
    # frame 값 지정
    browser.switch_to.frame('main')

    # 팝업창 삭제
    try :
        browser.find_element((By.XPATH, "/html/body/div[4]/div[1]/button/span[1]")).click() # 추가
    except :
        pass

    # 리스트에 없는 과목 예외처리
    try :
        searching = browser.find_element(By.XPATH, i)
    except :
        print("모든 과제를 불러왔습니다.")
        break

    # 수강과목 클릭
    searching.click()

    # 수강과목 과제 클릭
    WebDriverWait(browser, 2).until(EC.element_to_be_clickable((By.XPATH, '//*[@id="3"]/ul/li[2]/a'))).click() # 추가


    # 수강과목 이름,과제내용 가져오기
    source = browser.page_source
    bs = bs4.BeautifulSoup(source, 'lxml')

    # 과제제목 크롤링
    titles = bs.find_all('h4','f14')
    # 제출기한 크롤링
    d_days = bs.find_all('table','boardListInfo')
    # 제출기한 시작과 끝 분할
    slice1 = slice(16)
    slice2 = slice(19, 35)
    # 과제내용 크롤링
    contents = bs.find_all('div','cont pb0')
    # 과목이름 크롤링
    course = bs.find('h1','f40')
    # 과제진행여부 크롤링 추가
    progresses = bs.find_all('span','f12')

    # 과제제목 저장, 과목이름 저장
    for title in titles:
        title_result.append(title.get_text().strip().replace("\t","").replace("\n","").replace("\xa0",""))
        course_result.append(course.get_text().replace("\t","").replace("\n","").replace("\xa0",""))

    # 제출기한 시작 저장
    for d_day_start in d_days:
        d_day_start_result.append(d_day_start.get_text().replace("\t","").replace("\n","").replace("\xa0","").replace("과제 정보 리스트제출기간점수공개일자연장제출제출여부평가점수","")[slice1])
    
    # 제출기한 끝 저장
    for d_day_end in d_days:
        d_day_end_result.append(d_day_end.get_text().replace("\t","").replace("\n","").replace("\xa0","").replace("과제 정보 리스트제출기간점수공개일자연장제출제출여부평가점수","")[slice2])
        
    # 제출여부 저장
    for clear in d_days:
        clear_result.append(clear.get_text().replace("\t","").replace("\n","").replace("\xa0","")
                                            .replace("과제 정보 리스트제출기간점수공개일자연장제출제출여부평가점수","")
                                            .replace("1","").replace("2","").replace("3","").replace("4","")
                                            .replace("5","").replace("6","").replace("7","").replace("8","")
                                            .replace("9","").replace("0","").replace("-","").replace(".","")
                                            .replace("~","").replace(":","").replace(" ","").replace("(","")
                                            .replace(")","").replace("미허용","").replace("허","").replace("용",""))
        
    # 과제내용 저장
    for content in contents:
        content_result.append(content.get_text().replace("\t","").replace("\n","").replace("\xa0",""))
    

    # 과제진행여부 저장 추가
    for progress in progresses:
        progress_result.append(progress.get_text().replace("\t","").replace("\n","").replace("\xa0",""))
    
    def getprogress(ch):
        if ch == "[진행중]" or ch == "[마감]" or ch == "[진행예정]":
            return True
        else:
            return None

    progress_result = list(filter(getprogress, progress_result)) 
      
    #Num_C = len(title_result)
    #Num = []
    # 딕셔너리 저자용 리스트 생성
    
    # 첫 화면으로 가기 위한 뒤로가기 두번
    browser.back()
    browser.back()

# json 파일용 딕셔너리 생성    

count = len(title_result)

a_dict = []
b_dict = {}

for j in range(count):
    dict_key.insert(j, 'tasks%d' %j)
    if progress_result[j] == "[진행중]" or progress_result[j] == "[진행예정]":
        for i in range(len(dict_key)):
            temp_dict = {"course" : course_result[i], "title" : title_result[i], "d_day_start" : d_day_start_result[i], "d_day_end" : d_day_end_result[i], "clear" : clear_result[i], "content" : content_result[i]}
        a_dict.append(temp_dict)
    else:
        pass

b_dict = {userid: a_dict}

with open('./'+ userid +'.json', 'w', encoding = "UTF-8") as f :
    json.dump(b_dict, f, ensure_ascii = False, default = str, indent = 4)

# 브라우저 종료
browser.quit()
```
현재 진행 상태: 크롤링을 통한 학교 홈페이지에서 로그인 후 수강 과목에 대한 과제 추출, 제출 기한, 제출 완료한 과제에 대해서는 포함하지 않고 Json 파일로 저장한다.
=> Json 파일로 저장하는 이유는 서버에게 전송하기 위해서이다.

---

### 피드백
1. 데이터 추출에 대한 시간이 오래 걸려 줄인다
2. 공지사항에 대한 웹 크로링 시도

---

### 피드백 후 첫번째 테스트
1. 각 기능별 모듈화로 코드 재구성
2. 멀티 프로세스를 이용한 접근
3. 모든 수강과목에 대한 동시 접근 -> 데이터 출력

```Python
# pip install webdriver-manager # 크롬드라이버 자동설치
# pip install selenium
# pip install bs4
# pip install lxml

from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
from selenium.common.exceptions import NoSuchElementException # 예외지정
from selenium.webdriver.chrome.service import Service
from selenium.common.exceptions import UnexpectedAlertPresentException as PE # 팝업 예외 지정
from webdriver_manager.chrome import ChromeDriverManager
from bs4 import BeautifulSoup as bs
from xml.dom.minidom import Element
from multiprocessing import Pool

import time
import bs4
import json
import re

# 로그인 정보
userid = "20211954"
password = "971126"

def getprogress(ch):
    if ch == "[진행중]" or ch == "[마감]" or ch == "[진행예정]":
        return True
    else:
        return None

# 과제 정보 가져오기
def subject(i):
    # 과제에 대한 리스트 선언
    title_result = []
    d_day_start_result = []
    d_day_end_result = []
    content_result = []
    d_end_result = []
    course_result = []
    clear_result = []
    progress_result = []

    # json 리스트 선언
    dict_key = []
    temp_dict = {}

    # 현재 수강 과목 리스트로 저장 (최대 8개)
    subject_list = ['//*[@id="mCSB_1_container"]/li[1]/a/span[1]',
                    '//*[@id="mCSB_1_container"]/li[2]/a/span[1]',
                    '//*[@id="mCSB_1_container"]/li[3]/a/span[1]',
                    '//*[@id="mCSB_1_container"]/li[4]/a/span[1]',
                    '//*[@id="mCSB_1_container"]/li[5]/a/span[1]',
                    '//*[@id="mCSB_1_container"]/li[6]/a/span[1]',
                    '//*[@id="mCSB_1_container"]/li[7]/a/span[1]',
                    '//*[@id="mCSB_1_container"]/li[8]/a/span[1]']

    chrome_options = webdriver.ChromeOptions()

    # 크롤링으로 인한 사이트 차단을 막음 추가
    user_agent = "Mozilla/5.0 (Linux; Android 9; SM-G975F) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/71.0.3578.83 Mobile Safari/537.36" # 추가
    chrome_options.add_argument('user-agent=' + user_agent) # 추가

    # 브라우저 창 없이 실행
    chrome_options.add_argument("--headless")
    chrome_options.add_argument("--window-size=1920,1080")
    chrome_options.add_argument("--disable-gpu")  # 추가
    chrome_options.add_experimental_option('excludeSwitches', ['enable-logging'])

    # Chromedriver 경로 설정
    browser = webdriver.Chrome(service = Service(ChromeDriverManager().install()), options = chrome_options)
    # browser.maximize_window() # 전체 화면 설정

    # url 이동
    browser.get("https://lms.gwnu.ac.kr/Main.do?cmd=viewHome&userDTO.localeKey=ko")  # 변경

    # id, pw 입력 기존 방식과 다른 붙여넣기 방식으로 입력
    browser.execute_script("arguments[0].value=arguments[1]", browser.find_element(By.ID, "id"), userid) # 추가
    browser.execute_script("arguments[0].value=arguments[1]", browser.find_element(By.ID, "pw"), password) # 추가

    # 로그인 버튼 클릭
    WebDriverWait(browser, 5).until(EC.element_to_be_clickable((By.XPATH, "//*[@id='loginForm']/fieldset/p[2]/a"))).click() # 변경

    # 팝업창 삭제
    try :
        browser.find_element((By.XPATH, "/html/body/div[4]/div[1]/button/span[1]")).click() # 추가
    except :
        pass

    # 리스트에 없는 과목 예외처리
    try :
        browser.find_element(By.XPATH, subject_list[i])
    except :
        # 브라우저 종료
        browser.quit()
        return None

    # 수강과목 클릭
    browser.find_element(By.XPATH, subject_list[i]).click()

    try : 
        WebDriverWait(browser, 0.2).until(EC.alert_is_present())
        alert = browser.switch_to.alert

        alert.dismiss()

        alert.accept()

    except :        
            # 수강과목 과제 클릭
            WebDriverWait(browser, 10).until(EC.element_to_be_clickable((By.XPATH, '//*[@id="3"]/ul/li[2]/a'))).click() # 추가

            # 수강과목 이름,과제내용 가져오기
            source = browser.page_source
            bs = bs4.BeautifulSoup(source, 'lxml')

            # 과제제목 크롤링
            titles = bs.find_all('h4','f14')
            # 제출기한 크롤링
            d_days = bs.find_all('table','boardListInfo')
            # 제출기한 시작과 끝 분할
            slice1 = slice(16)
            slice2 = slice(19, 35)
            # 과제내용 크롤링
            contents = bs.find_all('div','cont pb0')
            # 과목이름 크롤링
            course = bs.find('h1','f40')
            # 과제진행여부 크롤링 추가
            progresses = bs.find_all('span','f12')

            # 과제제목 저장, 과목이름 저장
            for title in titles:
                title_result.append(title.get_text().strip().replace("\t","").replace("\n","").replace("\xa0",""))
                course_result.append(course.get_text().replace("\t","").replace("\n","").replace("\xa0",""))

            # 제출기한 시작 저장
            for d_day_start in d_days:
                d_day_start_result.append(d_day_start.get_text().replace("\t","").replace("\n","").replace("\xa0","").replace("과제 정보 리스트제출기간점수공개일자연장제출제출여부평가점수","")[slice1])
            
            # 제출기한 끝 저장
            for d_day_end in d_days:
                d_day_end_result.append(d_day_end.get_text().replace("\t","").replace("\n","").replace("\xa0","").replace("과제 정보 리스트제출기간점수공개일자연장제출제출여부평가점수","")[slice2])
                
            # 제출여부 저장
            for clear in d_days:
                clear_result.append(clear.get_text().replace("\t","").replace("\n","").replace("\xa0","")
                                                    .replace("과제 정보 리스트제출기간점수공개일자연장제출제출여부평가점수","")
                                                    .replace("1","").replace("2","").replace("3","").replace("4","")
                                                    .replace("5","").replace("6","").replace("7","").replace("8","")
                                                    .replace("9","").replace("0","").replace("-","").replace(".","")
                                                    .replace("~","").replace(":","").replace(" ","").replace("(","")
                                                    .replace(")","").replace("미허용","").replace("허","").replace("용",""))
                
            # 과제내용 저장
            for content in contents:
                content_result.append(content.get_text().replace("\t","").replace("\n","").replace("\xa0",""))
            

            # 과제진행여부 저장 추가
            for progress in progresses:
                progress_result.append(progress.get_text().replace("\t","").replace("\n","").replace("\xa0",""))

            progress_result = list(filter(getprogress, progress_result)) 

            count = len(title_result)

            for i in range(count):
                if progress_result[i] == "[진행중]" or progress_result[i] == "[진행예정]":
                    temp_dict = {"course" : course_result[i], "title" : title_result[i], "d_day_start" : d_day_start_result[i], 
                                 "d_day_end" : d_day_end_result[i], "clear" : clear_result[i], "content" : content_result[i]}
                    dict_key.append(temp_dict)
                else:
                    pass

            return dict_key

if __name__ == '__main__':
    start_time = time.time()

    p = Pool(8)
    num_list = [1,2,3,4,5,6,7,8]

    # 멀티프로세서로 수강과목의 과제들을 리턴 받음
    temp_dicts = p.map(subject, num_list)

    # 리턴한 값 중 None 값을 제거
    temp_dicts = list(filter(None, temp_dicts))

    # 2차원 리스트를 1차원으로 만들어줌
    temp_dicts2 = sum(temp_dicts,[])

    # json 딕셔너리 형식으로 저장
    b_dict = {"tasks": temp_dicts2}
    
    with open('./'+ userid +'.json', 'w', encoding = "UTF-8") as f :
        json.dump(b_dict, f, ensure_ascii = False, default = str, indent = 4)

    print("모든 과제를 불러왔습니다.")
    print(time.time() - start_time)
    
    p.close()
    p.join()
```

### 피드백
1. 멀티 프로세스Pool()을 이용한 여러개의 스레드로 크롤링 진행
2. 시간이 단축 되었지만 아직 부족한 상황
3. CPU 성능에 따라 스레드 활용성이 달라 몇개의 스레드가 제일 효율이 높은지 테스트

```Python
# pip install webdriver-manager # 크롬드라이버 자동설치
# pip install selenium
# pip install bs4
# pip install lxml

from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
from selenium.common.exceptions import NoSuchElementException # 예외지정
from selenium.webdriver.chrome.service import Service
from selenium.common.exceptions import UnexpectedAlertPresentException as PE # 팝업 예외 지정
from webdriver_manager.chrome import ChromeDriverManager
from bs4 import BeautifulSoup as bs
from xml.dom.minidom import Element

import time
import bs4
import json
import re
import sys

start_time = time.time()

chrome_options = webdriver.ChromeOptions()

# 브라우저 창 없이 실행
chrome_options.add_argument("--headless")
chrome_options.add_argument("--window-size=1920,1080")
chrome_options.add_argument("--disable-gpu") 
chrome_options.add_experimental_option('excludeSwitches', ['enable-logging'])
# 오류제어 추가
chrome_options.add_argument("--no-sandbox")  # 대부분의 리소스에 대한 액세스를 방지 추가
chrome_options.add_argument("--disable-setuid-sandbox") # 크롬 충돌을 막아줌 추가 
chrome_options.add_argument("--disable-dev-shm-usage") # 메모리가 부족해서 에러가 발생하는 것을 막아줌 추가

# Chromedriver 경로 설정
browser = webdriver.Chrome(service = Service(ChromeDriverManager().install()), options = chrome_options)
# browser.maximize_window() # 전체 화면 설정

# 로그인 정보
userid = "20161854"
password = "shgustjq12!"

# url 이동
browser.get("https://lms.gwnu.ac.kr/Main.do?cmd=viewHome&userDTO.localeKey=ko")  # 변경

# id, pw 입력 기존 방식과 다른 붙여넣기 방식으로 입력
browser.execute_script("arguments[0].value=arguments[1]", browser.find_element(By.ID, "id"), userid) # 추가
browser.execute_script("arguments[0].value=arguments[1]", browser.find_element(By.ID, "pw"), password) # 추가

# 로그인 버튼 클릭
WebDriverWait(browser, 3).until(EC.element_to_be_clickable((By.XPATH, "//*[@id='loginForm']/fieldset/p[2]/a"))).click() # 변경

# 계정 정보가 일치하지 않을 시 예외처리 추가
alert_result = ""

try :
    WebDriverWait(browser, 0.1).until(EC.alert_is_present()) 
    alert = browser.switch_to.alert
    alert_result = alert.text
    alert.accept()
except :
    pass

if alert_result == "입력하신 아이디 혹은 비밀번호가 일치하지 않습니다." :
    print("입력하신 아이디 혹은 비밀번호가 일치하지 않습니다.")
    # 계정정보가 동일하지 않을 시 파이썬 종료
    browser.quit()
    sys.exit(0)

else :
    # 현재 수강 과목 리스트로 저장 (최대 8개)
    subject_list = ['//*[@id="mCSB_1_container"]/li[1]/a/span[1]',
                    '//*[@id="mCSB_1_container"]/li[2]/a/span[1]',
                    '//*[@id="mCSB_1_container"]/li[3]/a/span[1]',
                    '//*[@id="mCSB_1_container"]/li[4]/a/span[1]',
                    '//*[@id="mCSB_1_container"]/li[5]/a/span[1]',
                    '//*[@id="mCSB_1_container"]/li[6]/a/span[1]',
                    '//*[@id="mCSB_1_container"]/li[7]/a/span[1]',
                    '//*[@id="mCSB_1_container"]/li[8]/a/span[1]']


    # 과제에 대한 리스트 선언
    title_result = []
    d_day_start_result = []
    d_day_end_result = []
    content_result = []
    d_end_result = []
    course_result = []
    clear_result = []
    progress_result = []
    professor_result = [] 
    extend_result = [] # 연장제출기한 추가

    # json 리스트 선언
    dict_key = []
    temp_dict = {}

    a_dict = []
    b_dict = {}

    # 과제 정보 가져오기
    for i in subject_list :

        # 팝업창 삭제
        try :
            browser.find_element((By.XPATH, "/html/body/div[4]/div[1]/button/span[1]")).click() # 추가
        except :
            pass

        # 리스트에 없는 과목 예외처리
        try :
            searching = browser.find_element(By.XPATH, i)
        except :
            print("모든 과제를 불러왔습니다.")
            break

        # 수강과목 클릭
        searching.click()

        try : 
            WebDriverWait(browser, 0.1).until(EC.alert_is_present())
            alert = browser.switch_to.alert

            alert.accept()
            continue

        except :        
            try :
                # 수강과목 과제 클릭
                WebDriverWait(browser, 2).until(EC.element_to_be_clickable((By.XPATH, '//*[@id="3"]/ul/li[2]/a'))).click() # 추가

                # 수강과목 이름,과제내용 가져오기
                source = browser.page_source
                bs = bs4.BeautifulSoup(source, 'lxml')

                # 과제제목 크롤링
                titles = bs.find_all('h4','f14')
                # 제출기한 크롤링
                d_days = bs.find_all('table','boardListInfo')
                # 제출기한 시작과 끝 분할
                slice1 = slice(16)
                slice2 = slice(19, 35)
                # 과제내용 크롤링
                contents = bs.find_all('div','cont pb0')
                # 과목이름 크롤링
                course = bs.find('h1','f40')
                # 교수명 크롤링
                professor = bs.select_one('#headerContent > div > ul.postCover > li.tinfoList > table > tbody > tr > td.first')
                # 과제진행여부 크롤링
                progresses = bs.find_all('span','f12')
                # 연장제출기간 추가
                extends = bs.select('table.boardListInfo > tbody > tr > td:nth-child(3)')

                # 과제제목 저장, 과목이름 저장, 교수명 저장
                for title in titles:
                    title_result.append(title.get_text().strip().replace("\t","").replace("\n","").replace("\xa0",""))
                    course_result.append(course.get_text().replace("\t","").replace("\n","").replace("\xa0",""))
                    professor_result.append(professor.get_text().replace("\t","").replace("\n","").replace("\xa0","").replace(" ","")) # 교수명 크롤링 추가

                # 제출기한 시작 저장
                for d_day_start in d_days:
                    d_day_start_result.append(d_day_start.get_text().replace("\t","").replace("\n","").replace("\xa0","").replace("과제 정보 리스트제출기간점수공개일자연장제출제출여부평가점수","")[slice1])
                
                # 제출기한 끝 저장
                for d_day_end in d_days:
                    d_day_end_result.append(d_day_end.get_text().replace("\t","").replace("\n","").replace("\xa0","").replace("과제 정보 리스트제출기간점수공개일자연장제출제출여부평가점수","")[slice2])
                    
                # 제출여부 저장
                for clear in d_days:
                    clear_result.append(clear.get_text().replace("\t","").replace("\n","").replace("\xa0","")
                                                        .replace("과제 정보 리스트제출기간점수공개일자연장제출제출여부평가점수","")
                                                        .replace("1","").replace("2","").replace("3","").replace("4","")
                                                        .replace("5","").replace("6","").replace("7","").replace("8","")
                                                        .replace("9","").replace("0","").replace("-","").replace(".","")
                                                        .replace("~","").replace(":","").replace(" ","").replace("(","")
                                                        .replace(")","").replace("미허용","").replace("허","").replace("용",""))
                    
                # 과제내용 저장
                for content in contents:
                    content_result.append(content.get_text().replace("\t","").replace("\n","").replace("\xa0",""))
                

                # 과제진행여부 저장 
                for progress in progresses:
                    progress_result.append(progress.get_text().replace("\t","").replace("\n","").replace("\xa0",""))
                
                def getprogress(ch):
                    if ch == "[진행중]" or ch == "[마감]" or ch == "[진행예정]" or ch == "[연장제출기간]":
                        return True
                    else:
                        return None

                progress_result = list(filter(getprogress, progress_result)) 
                
                # 연장제출기간 추가
                for extend in extends:
                    extend_result.append(extend.get_text().replace("\t","").replace("\n","").replace("\xa0","").replace("허 용","").replace("미허용","").replace("(","").replace(")","").lstrip())
            
                # 첫 화면으로 가기 위한 뒤로가기 두번
                browser.back()
                browser.back()
            except :
                continue
            continue

    # 브라우저 종료
    browser.quit()

    # json 파일용 딕셔너리 생성 교수명 추가
    count = len(title_result)

    for j in range(count):
        dict_key.insert(j, 'tasks%d' %j)
        if progress_result[j] == "[진행중]" or progress_result[j] == "[진행예정]":
            for i in range(len(dict_key)):
                temp_dict = {"course" : course_result[i], "professor" : professor_result[i], "title" : title_result[i], "d_day_start" : d_day_start_result[i], "d_day_end" : d_day_end_result[i], "clear" : clear_result[i], "content" : content_result[i]}
            a_dict.append(temp_dict)
        elif progress_result[j] == "[연장제출기간]" :   # 추가  
            for i in range(len(dict_key)):
                temp_dict = {"course" : course_result[i], "professor" : professor_result[i], "title" : title_result[i], "d_day_start" : d_day_start_result[i], "d_day_end" : extend_result[i], "clear" : clear_result[i], "content" : content_result[i]}
            a_dict.append(temp_dict)
        else :
            pass

    b_dict = {userid: a_dict}

    with open('./'+ userid +'.json', 'w', encoding = "UTF-8") as f :
        json.dump(b_dict, f, ensure_ascii = False, default = str, indent = 4)
    
    if a_dict == "[]" :
        print("과제를 불러오기 실패했거나 과제가 존재하지 않습니다.")

print(time.time() - start_time)
```

### 피드백
1. 멀티 프로세스를 사용하려 하였으나 기기 성능에 따른 속도 차이 문제, chromedriver가 모든 프로세스에 설치되어 있어야지만 실행 가능하다는 점의 문제점 발견
2. 결국 크롤링을 통한 데이터 추출은 새벽 시간대에 한 후 DB에 저장하여 업데이트
3. 교수님 성함, 과제 이름에 따른 조건문 변경, 제출 기간이 연장되었을 경우 연장된 기간 출력

```Python
# pip install webdriver-manager # 크롬드라이버 자동설치
# pip install selenium
# pip install bs4
# pip install lxml

from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
from selenium.common.exceptions import NoSuchElementException # 예외지정
from selenium.webdriver.chrome.service import Service
from selenium.common.exceptions import UnexpectedAlertPresentException as PE # 팝업 예외 지정
from webdriver_manager.chrome import ChromeDriverManager
from bs4 import BeautifulSoup as bs
from xml.dom.minidom import Element

import time
import bs4
import json
import re
import sys

start_time = time.time()

chrome_options = webdriver.ChromeOptions()

# 브라우저 창 없이 실행
#chrome_options.add_argument("--headless")
chrome_options.add_argument("--window-size=1920,1080")
chrome_options.add_argument("--disable-gpu") 
chrome_options.add_experimental_option('excludeSwitches', ['enable-logging'])
# 오류제어 추가
chrome_options.add_argument("--no-sandbox")  # 대부분의 리소스에 대한 액세스를 방지 추가
chrome_options.add_argument("--disable-setuid-sandbox") # 크롬 충돌을 막아줌 추가 
chrome_options.add_argument("--disable-dev-shm-usage") # 메모리가 부족해서 에러가 발생하는 것을 막아줌 추가

# Chromedriver 경로 설정
browser = webdriver.Chrome(service = Service(ChromeDriverManager().install()), options = chrome_options)
# browser.maximize_window() # 전체 화면 설정

# 로그인 정보
userid = "20171454"
password = "whddnjs123@"

# url 이동
browser.get("https://lms.gwnu.ac.kr/Main.do?cmd=viewHome&userDTO.localeKey=ko")  # 변경

# id, pw 입력 기존 방식과 다른 붙여넣기 방식으로 입력
browser.execute_script("arguments[0].value=arguments[1]", browser.find_element(By.ID, "id"), userid) # 추가
browser.execute_script("arguments[0].value=arguments[1]", browser.find_element(By.ID, "pw"), password) # 추가

# 로그인 버튼 클릭
WebDriverWait(browser, 3).until(EC.element_to_be_clickable((By.XPATH, "//*[@id='loginForm']/fieldset/p[2]/a"))).click() # 변경

# 계정 정보가 일치하지 않을 시 예외처리 추가
alert_result = ""

try :
    WebDriverWait(browser, 0.1).until(EC.alert_is_present()) 
    alert = browser.switch_to.alert
    alert_result = alert.text
    alert.accept()
except :
    pass

if alert_result == "입력하신 아이디 혹은 비밀번호가 일치하지 않습니다." :
    print("입력하신 아이디 혹은 비밀번호가 일치하지 않습니다.")
    # 계정정보가 동일하지 않을 시 파이썬 종료
    browser.quit()
    sys.exit(0)

else :
    # 현재 수강 과목 리스트로 저장 (최대 8개)
    subject_list = ['//*[@id="mCSB_1_container"]/li[1]/a/span[1]',
                    '//*[@id="mCSB_1_container"]/li[2]/a/span[1]',
                    '//*[@id="mCSB_1_container"]/li[3]/a/span[1]',
                    '//*[@id="mCSB_1_container"]/li[4]/a/span[1]',
                    '//*[@id="mCSB_1_container"]/li[5]/a/span[1]',
                    '//*[@id="mCSB_1_container"]/li[6]/a/span[1]',
                    '//*[@id="mCSB_1_container"]/li[7]/a/span[1]',
                    '//*[@id="mCSB_1_container"]/li[8]/a/span[1]']


    # 과제에 대한 리스트 선언
    title_result = []
    d_day_start_result = []
    d_day_end_result = []
    content_result = []
    d_end_result = []
    course_result = []
    clear_result = []
    progress_result = []
    professor_result = [] 
    extend_result = [] # 연장제출기한 추가

    # json 리스트 선언
    dict_key = []
    temp_dict = {}

    a_dict = []
    b_dict = {}

    # 과제 정보 가져오기
    for i in subject_list :

        # 팝업창 삭제
        try :
            browser.find_element((By.XPATH, "/html/body/div[4]/div[1]/button/span[1]")).click() # 추가
        except :
            pass

        # 리스트에 없는 과목 예외처리
        try :
            searching = browser.find_element(By.XPATH, i)
        except :
            print("모든 과제를 불러왔습니다.")
            break

        # 수강과목 클릭
        searching.click()

        try : 
            WebDriverWait(browser, 0.1).until(EC.alert_is_present())
            alert = browser.switch_to.alert

            alert.accept()
            continue

        except :        
            try :
                # 수강과목 과제 클릭
                WebDriverWait(browser, 2).until(EC.element_to_be_clickable((By.XPATH, '//*[@id="3"]/ul/li[2]/a'))).click() # 추가

                # 수강과목 이름,과제내용 가져오기
                source = browser.page_source
                bs = bs4.BeautifulSoup(source, 'lxml')

                # 과제제목 크롤링
                titles = bs.find_all('h4','f14')
                # 제출기한 크롤링
                d_days = bs.find_all('table','boardListInfo')
                # 제출기한 시작과 끝 분할
                slice1 = slice(16)
                slice2 = slice(19, 35)
                # 과제내용 크롤링
                contents = bs.find_all('div','cont pb0')
                # 과목이름 크롤링
                course = bs.find('h1','f40')
                # 교수명 크롤링
                professor = bs.select_one('#headerContent > div > ul.postCover > li.tinfoList > table > tbody > tr > td.first')
                # 과제진행여부 크롤링
                progresses = bs.find_all('span','f12')
                # 연장제출기간 추가
                extends = bs.select('table.boardListInfo > tbody > tr > td:nth-child(3)')

                # 과제제목 저장, 과목이름 저장, 교수명 저장
                for title in titles:
                    title_result.append(title.get_text().strip().replace("\t","").replace("\n","").replace("\xa0",""))
                    course_result.append(course.get_text().replace("\t","").replace("\n","").replace("\xa0",""))
                    professor_result.append(professor.get_text().replace("\t","").replace("\n","").replace("\xa0","").replace(" ","")) # 교수명 크롤링 추가

                # 제출기한 시작 저장
                for d_day_start in d_days:
                    d_day_start_result.append(d_day_start.get_text().replace("\t","").replace("\n","").replace("\xa0","").replace("과제 정보 리스트제출기간점수공개일자연장제출제출여부평가점수","")[slice1])
                
                # 제출기한 끝 저장
                for d_day_end in d_days:
                    d_day_end_result.append(d_day_end.get_text().replace("\t","").replace("\n","").replace("\xa0","").replace("과제 정보 리스트제출기간점수공개일자연장제출제출여부평가점수","")[slice2])
                    
                # 제출여부 저장
                for clear in d_days:
                    clear_result.append(clear.get_text().replace("\t","").replace("\n","").replace("\xa0","")
                                                        .replace("과제 정보 리스트제출기간점수공개일자연장제출제출여부평가점수","")
                                                        .replace("1","").replace("2","").replace("3","").replace("4","")
                                                        .replace("5","").replace("6","").replace("7","").replace("8","")
                                                        .replace("9","").replace("0","").replace("-","").replace(".","")
                                                        .replace("~","").replace(":","").replace(" ","").replace("(","")
                                                        .replace(")","").replace("미허용","").replace("허","").replace("용",""))
                    
                # 과제내용 저장
                for content in contents:
                    content_result.append(content.get_text().replace("\t","").replace("\n","").replace("\xa0",""))
                

                # 과제진행여부 저장 
                for progress in progresses:
                    progress_result.append(progress.get_text().replace("\t","").replace("\n","").replace("\xa0",""))
                
                def getprogress(ch):
                    if ch == "[진행중]" or ch == "[마감]" or ch == "[진행예정]" or ch == "[연장제출기간]":
                        return True
                    else:
                        return None

                progress_result = list(filter(getprogress, progress_result)) 
                
                # 연장제출기간 추가
                for extend in extends:
                    extend_result.append(extend.get_text().replace("\t","").replace("\n","").replace("\xa0","").replace("허 용","").replace("미허용","").replace("(","").replace(")","").lstrip())
            
                # 첫 화면으로 가기 위한 뒤로가기 두번
                browser.back()
                browser.back()
            except :
                continue
            continue

    # 브라우저 종료
    browser.quit()

    # json 파일용 딕셔너리 생성 교수명 추가
    count = len(title_result)

    for j in range(count):
        dict_key.insert(j, 'tasks%d' %j)
        if progress_result[j] == "[진행중]" or progress_result[j] == "[진행예정]":
            for i in range(len(dict_key)):
                temp_dict = {"course" : course_result[i], "professor" : professor_result[i], "title" : title_result[i], "d_day_start" : d_day_start_result[i], "d_day_end" : d_day_end_result[i], "clear" : clear_result[i], "content" : content_result[i]}
            a_dict.append(temp_dict)
        elif progress_result[j] == "[연장제출기간]" :   # 추가  
            for i in range(len(dict_key)):
                temp_dict = {"course" : course_result[i], "professor" : professor_result[i], "title" : title_result[i], "d_day_start" : d_day_start_result[i], "d_day_end" : extend_result[i], "clear" : clear_result[i], "content" : content_result[i]}
            a_dict.append(temp_dict)
        else :
            pass

    b_dict = {userid: a_dict}

    with open('./'+ userid +'.json', 'w', encoding = "UTF-8") as f :
        json.dump(b_dict, f, ensure_ascii = False, default = str, indent = 4)
    
    if a_dict == "[]" :
        print("과제를 불러오기 실패했거나 과제가 존재하지 않습니다.")

print(time.time() - start_time)
```

### 최종 결과
1. 학교 웹 페이지 접속 -> 로그인 -> 수강과목 -> 과제란 -> 데이터 추출
2. 데이터 추출 시간을 감안하여 새벽 시간대 미리 DB에 저장 후 스케쥴링을 통해 APP로 데이터 출력

### 느낀점

크롤링 파트를 맡아 같은 팀원과 함께 수행하여 만든 코드가 서버에서 오류가 발생하여 이를 보안하고, 각 분야별 팀원들과 작업하는 도중 중간중간 발생하는 오류, 기록하지는 못했지만 예상하지 못한 오류 등 생각 보다 많은 문제점을 같이 해결해 나아가며 많은 학습이 있는 기회 였다고 생각합니다. 중간에 발생한 오류들과 해결 과정을 상세하게 기록하지 못한것이 아쉽지만 2학기에 있을 프로젝트에서 더욱 자세하게 기록하며 발전해 나아가겠습니다:)