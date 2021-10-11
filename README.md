# 20150709_강수인_네이버 지도 크롤링
20150709_강수인

# -*- coding: utf8 -*-
import re
import requests
import csv

from selenium import webdriver
from selenium.common.exceptions import NoSuchElementException

from selenium.webdriver import Chrome
from selenium.webdriver.common.by import By
from selenium.webdriver.common.keys import Keys
from selenium.webdriver.support import expected_conditions as EC
from selenium.webdriver.support.ui import WebDriverWait

import datetime as dt
import time
from IPython.display import display

from openpyxl import load_workbook
import openpyxl as O
from openpyxl.workbook import workbook
import pandas as pd
from bs4 import BeautifulSoup

options = webdriver.ChromeOptions()
options.add_experimental_option("excludeSwitches", ["enable-logging"])
browser = webdriver.Chrome(options=options)

headers = {"User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/94.0.4606.71 Safari/537.36"}

# 검색어 
keyword = "서대문 스타벅스"

# url패턴으로 타겟 url로 접속해 html구조 알아내기
url = "https://m.map.naver.com/search2/search.naver?query=" + keyword
browser.get(url)
browser.implicitly_wait(10)

Excel_file = r"C:\Users\stead\PythonWorkspace\naver_maps_crawling.xlsx"
wb = O.load_workbook(Excel_file)

# 지점 클릭
for i in range(2,3):
    try:
        restaurant = browser.find_element_by_xpath("//*[@id='ct']/div[2]/ul/li[{}]/div[1]/a/div/strong".format(i))
        restaurant.click()
        browser.implicitly_wait(10)

        # 리뷰 클릭
        review = browser.find_element_by_link_text("리뷰")
        review.click()
        browser.implicitly_wait(10)

        start = dt.datetime.now()
        end = start + dt.timedelta(seconds=500)

        # 스크롤 내리기(500초)
        while True:
            # 현재 문서 높이를 가져와서 저장
            prev_height = browser.execute_script("return document.body.scrollHeight")

            interval = 2
            # 스크롤을 가장 아래로 내림
            browser.execute_script("window.scrollTo(0, document.body.scrollHeight)")

            # 페이지 로딩 대기
            time.sleep(interval)

            try: 
                more_review = browser.find_element_by_link_text("더보기")
                more_review.click()
            except:
                break

            # 현재 문서 높이를 가져와서 저장
            curr_height = browser.execute_script("return document.body.scrollHeight")
            if dt.datetime.now() > end:
                break

        # 크롤링
        html = browser.page_source  # 접속한 url의 html 소스 가져오기 (단, 로딩된 데이터만!)
        soup = BeautifulSoup(html, 'html.parser')  # BeautifulSoup으로 html구조 파싱

        cleaner = re.compile('<.*?>')
        df = pd.DataFrame(columns=("date",'Review_contents','Star_ratings'))
        
        informations = soup.find_all("li", attrs= {"class":"_2Cv-r"})
        for information in informations:
           
            a = information.find_all("span", attrs={"class": "_3WqoL"})
            b = information.find_all("span", attrs={"class": "WoYOw"})
            c = information.find_all("span", attrs={"class": "_2tObC"})
            
            cleaning = re.sub(cleaner,"",str(a))
            date = re.findall(r'\d{4}.\d{2}.\d{2}', cleaning)

            try: Review_content = information.select('span.WoYOw')[0].text 
            except: Review_content = '리뷰 없음'

            Star_rating = information.select('span._2tObC')[0].text 
            df = df.append({"date": date,'Review_contents': Review_content,'Star_ratings': Star_rating},ignore_index=True)
            
        df.reset_index(inplace=True)
        display(pd.DataFrame(df))

        df.to_excel("naver_maps_crawling.xlsx",index=False)

        browser.back()
        browser.back()

    except NoSuchElementException: # 사진 없는 경우
        pass          
        restaurant = browser.find_element_by_xpath("//*[@id='ct']/div[2]/ul/li[{}]/div[1]/a/div/strong".format(i))
        restaurant.click()
        browser.implicitly_wait(10)

        # 리뷰 클릭
        review = browser.find_element_by_link_text("리뷰")
        review.click()
        browser.implicitly_wait(10)

        start = dt.datetime.now()
        end = start + dt.timedelta(seconds=500)

        # 스크롤 내리기(60초)
        while True:
            # 현재 문서 높이를 가져와서 저장
            prev_height = browser.execute_script("return document.body.scrollHeight")

            interval = 2
            # 스크롤을 가장 아래로 내림
            browser.execute_script("window.scrollTo(0, document.body.scrollHeight)")

            # 페이지 로딩 대기
            time.sleep(interval)

            try: 
                more_review = browser.find_element_by_link_text("더보기")
                more_review.click()
            except:
                break

            # 현재 문서 높이를 가져와서 저장
            curr_height = browser.execute_script("return document.body.scrollHeight")
            if dt.datetime.now() > end:
                break

        # 크롤링
        html = browser.page_source  # 접속한 url의 html 소스 가져오기 (단, 로딩된 데이터만!)
        soup = BeautifulSoup(html, 'html.parser')  # BeautifulSoup으로 html구조 파싱

        cleaner = re.compile('<.*?>')
        df = pd.DataFrame(columns=("date",'Review_contents','Star_ratings'))
        
        informations = soup.find_all("li", attrs= {"class":"_2Cv-r"})
        for information in informations:
           
            a = information.find_all("span", attrs={"class": "_3WqoL"})
            b = information.find_all("span", attrs={"class": "WoYOw"})
            c = information.find_all("span", attrs={"class": "_2tObC"})
            
            cleaning = re.sub(cleaner,"",str(a))
            date = re.findall(r'\d{4}.\d{2}.\d{2}', cleaning)

            try: Review_content = information.select('span.WoYOw')[0].text 
            except: Review_content = '리뷰 없음'

            Star_rating = information.select('span._2tObC')[0].text 
            df = df.append({"date": date,'Review_contents': Review_content,'Star_ratings': Star_rating},ignore_index=True)
            
        df.reset_index(inplace=True)
        display(pd.DataFrame(df))

        df.to_excel("naver_maps_crawling.xlsx",index=False)

        browser.back()
        browser.back()
