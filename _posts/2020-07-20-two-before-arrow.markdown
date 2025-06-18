---
layout: post
title:  "lee의 기술 블로그"
date:   2025-05-15
category: development
image: assets/img/t.webp
author: Lee
tags: lee
comments: true
---

lee의 기술 블로그에 오신걸 환영합니다.

2025.05.15


지난 블로그에서는 간단한 블로그 소개를 하였다.

이번주 블로그에서는 암호화 데이터 client.py, server.py, linear_model.py, 
handler.py 를 생성하여 평문을 암호화 하는 과정을 간단하게 소개를 할 것이다.
오늘 소개할 블로그는 client.py 이다.


.
.
.
.


client.py
: 동형암호 기반의 AI 예측 시스템에서 클라이언트 측 기능을 수행한다.       클라이언트가 입력 벡터를 암호화한 후 Flask 서버에 전송한 후, 서버로부터 암호화된 결과를 받아 복호화하는 구조로 되어 있다.

<code>

import base64
import requests
from crypto_utils import load_context, encrypt_vector, decrypt_vector 


context = load_context()
print("context scale:", context.global_scale) 


plain_vec = [2.0, 4.0, 8.0]
print("보낸 입력값:", plain_vec)


enc_vec = encrypt_vector(context, plain_vec)


enc_bytes = enc_vec.serialize()


enc_b64 = base64.b64encode(enc_bytes).decode("utf-8")


response = requests.post("http://127.0.0.1:5000/predict", json={"enc_input": enc_b64})

try:
    response_json = response.json()
    
 if "enc_result" in response_json:
  result_b64 = response_json["enc_result"]  
  result_bytes = base64.b64decode(result_b64)  
  enc_result = decrypt_vector(context, result_bytes) 
    print("복호화된 결과:", enc_result.decrypt())  
  else:
   print("서버 응답 오류:", response_json.get("error", "알 수 없는 오류"))

except Exception as e:
    print("❗ 예외 발생:", e)
    print("📨 서버 응답 원본:", response.text)


< 이번엔 코드를 분석해보자 >

import base64
-> base64 모듈, python 표준 라이브러리에 포함된 모듈이다.
   이 모듈은 바이너리 데이터를 텍스트로 인코딩 & 디코딩 할 때 사용한다.
   텍스트 기반(HTTP & JSON) 통신에서는 순수 바이너리 데이터를 보낼 수가 없기 때문에, 암호화된 데이터를 Base64로 인코딩해서 전송.

import requests
-> requests 모듈, python에서 HTTP 요청을 보낼 수 있게 해주는 외부            
   라이브러리이다. 주로 웹 서버랑 통신할 때 사용한다. 

from crypto_utils import load_context, encrypt_vector, decrypt_vector
-> crypto_utils.py 파일(모듈) 안에 있는 load_context,       encrypt_vector, decrypt_vector 라는 세 개의 함수만 선택하겠다는 의미이다.

context = load_context()
-> load_context() 함수를 호출해서 암호화 환경(context) 만든 후, context 변수에 저장한다. 

print("context scale:", context.global_scale)
-> 결과 값

plain_vec = [2.0, 4.0, 8.0]
-> plain_vec 변수에 실수형 숫자 3개로 이루어진 리스트에 저장한다. 암호화하기 전의 평문 벡터(Plaintext Vector)

print("보낸 입력값:", plain_vec)
-> 결과 값

enc_vec = encrypt_vector(context, plain_vec)
-> plain_vec 라는 평문 벡터를 context 설정을 이용하여 암호화한 후 결과를 enc_vec 라는 변수에 저장한다.


enc_bytes = enc_vec.serialize()
-> env_vec 라는 암호문 객체를 .serialize()를 이용해서 바이트(bytes) 형태로 변환한 후, 그걸 enc_bytes에 저장한다. 


enc_b64 = base64.b64encode(enc_bytes).decode("utf-8")
-> enc_bytes 라는 바이너리 데이터(암호문)을 Base64로 인코딩해서 문자열(str) 형태로 바꿈 코드이다.


response = requests.post("http://127.0.0.1:5000/predict", json={"enc_input": enc_b64})
-> 로컬에 있는 Flask 서버(127.0.0.1:5000)의 /predict 엔드포인트에 "enc_input": enc_b64 라는 데이터를 JSON 형식으로 보내는 POST 요청을 보낸다. 그 응답을 response 변수에 저장한다.

try:
 response_json = response.json()
 -> 서버 응답을 JSON 형식으로 한다.
    
if "enc_result" in response_json:
-> 응답에 "enc_result"  라는 키가 있는지 확인 후 결과가 제대로 왔는지 확인한다.

    result_b64 = response_json["enc_result"]  
    -> 암호화된 결과(base64 인코딩된 문자열)를 꺼낸다.

    result_bytes = base64.b64decode(result_b64)
    -> base64 문자열을 바이트 형태로 디코딩한다.

    enc_result = decrypt_vector(context, result_bytes) 
    -> 바이트를 다시 암호문 객체로 복원하고 복호화할 준비를 한다.
    decrypt_vector()는 내부적으로 ts.ckks_vector_from(context, result_bytes) 호출한다.

    print("복호화된 결과:", enc_result.decrypt())
    -> 출력  
else:
 print("서버 응답 오류:", response_json.get("error", "알 수 없는 오류"))
 -> 서버가 "enc_result" 안 했을 때, 오류 값 출력한다.

except Exception as e:
-> try 블록에서 오류가 생기면 이 블록이 실행된다. 
   e는 발생한 에러 객체를 담고 있다.

 print("❗ 예외 발생:", e)
 print("📨 서버 응답 원본:", response.text)
 -> 출력


.
.
.

client.py는 동형암호(CKKS)를 이용해서 데이터를 암호화한 뒤, 서버에 보내고, 서버의 암호 상태 연산 결과를 다시 받아 복호화하는 클라이언트 프로그램이라는 것을 알 수 있었다. 아직 코드를 이해하기엔 많이 미숙한 것 같다.

.
.
.

다음 블로그에서는 server.py 을 분석하여 소개할 것이다. 