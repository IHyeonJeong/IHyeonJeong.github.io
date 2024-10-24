---
layout: single
title: "오피스텔, 원룸의 데이터 분석 프로젝트 (1)"
typora-root-url: ../
categories: officetel
toc: true
author_profile: false
search: false
---

# 목표 및 문제 설정

<div class="notice--success">
    <ol> 
        <li> 문제 정의: 원룸과 오피스텔을 구할 때 내가 원하는 최적의 장소를 추천 </li>
        <li> 성과지표(KPI): 위치 및 편의성 지표<br>
            추천매물이 사용자 지정 위치(학교, 직장)까지 얼마나 가까운지, 어느정도 시간이    걸리는지 측정하고 주변에 편의점, 카페 등 편의 시설의 개수 측정 </li>
    </ol>
</div>

# 데이터 수집

## **2. 데이터 수집 단계**

### **목적**

- 목표 달성을 위해 필요한 데이터를 다양한 소스에서 수집합니다.

### **해야 할 일**

1. **데이터 소스 식별**

   - **웹 크롤링**: 부동산 매물 정보 등 웹에서 필요한 데이터 수집
   - **API 호출**: 공공 데이터 API, 소셜 미디어 API
   - **파일데이터**: 공공 기관에서 만든 파일

2. **수집 방법 결정**  
   각 데이터 소스에 맞는 수집 방법을 선택합니다.

   - **공공 데이터 API 수집**:

   ```python
   import requests
   import pandas as pd
   import xml.etree.ElementTree as ET
   ```

   ```python
   def fetch_public_data(service_key, gu_code, base_date):
   # 공공데이터포털 API URL 생성
   url = (
       "http://apis.data.go.kr/1613000/RTMSDataSvcOffiRent/getRTMSDataSvcOffiRent?"
       + "LAWD_CD=" + gu_code
       + "&DEAL_YMD=" + base_date
       + "&serviceKey=" + service_key
       + "&pageNo=1"
       + "&numOfRows=100"
   )
   print(url)

   # GET 요청 보내기
   response = requests.get(url)

   # 응답 확인 및 처리
   if response.status_code == 200:
       print("데이터를 성공적으로 가져왔습니다!")
       root = ET.fromstring(response.content)
       data = []
       for item in root.findall(".//item"):
           data.append({
               'buildYear': item.findtext("buildYear"),
               'dealYear': item.findtext("dealYear"),
               'dealMonth': item.findtext("dealMonth"),
               'dealDay': item.findtext("dealDay"),
               'deposit': item.findtext("deposit"),
               'excluUseAr': item.findtext("excluUseAr"),
               'floor': item.findtext("floor"),
               'jibun': item.findtext("jibun"),
               'monthlyRent': item.findtext("monthlyRent"),
               'offiNm': item.findtext("offiNm"),
               'sggCd': item.findtext("sggCd"),
               'sggNm': item.findtext("sggNm"),
               'umdNm': item.findtext("umdNm")
           })
       df = pd.DataFrame(data)
       return df
   else:
       print(f"오류 발생: {response.status_code}")
       return None
   ```

   ```python
   # 서비스 키와 지역 코드 설정
   service_key = "YOUR_SERVICE_KEY"
   ```

   - **로그 수집 및 데이터 스트리밍**: Kafka, MQTT 등을 활용한 실시간 데이터 수집

3. **데이터 품질 관리**  
   수집 단계에서 **데이터 품질**을 점검하고, 필요시 **정제**합니다.
   - **중복 데이터 제거**: 동일한 데이터가 여러 번 수집되지 않도록 관리
   - **누락 데이터 처리**: 결측치를 추적하고 보완하는 작업 수행
   - **데이터 포맷 정규화**: 수집된 데이터의 형식을 통일하여 저장
