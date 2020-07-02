---
layout: post
title: "Elasticsearch와 Kibana설치 방법"
excerpt: "Elasticsearch랑 kibana를 설치해보자"
header-img: img/post-bg/miui-ux.jpg
categories: [etc]
comments: true
---

# How to install elasticsearch and kibana 

- **Elasticsearch 설치**<br><br>


    Kibana를 사용하기 위해서는 Elasticsearch가 필수이며, 같은 버전을 사용해야 한다. 


    Elasticsearch는 Java 8 이상이 필요하다. <br><br>
    

    Java 설정이 확인되었다면, 아래 링크를 통해 elasticsearch 압축 파일을 다운받을 수 있다. 

    파일 다운로드: <https://www.elastic.co/kr/downloads/elasticsearch>


    파일 다운이 완료되면, 다음과 같은 명령어를 통해 압축을 푼다. 예시에서는 elasticsearch 7.6.1 버전을 사용했다. 

    ```
    tar -xvf elasticsearch-7.6.1.tar.gz
    ```


    다음과 같이 bin 디렉토리로 이동한다. 

    ```
    cd elasticsearch-7.6.1/bin
    ```


    클러스터 시작을 위해, 다음과 같은 명령어를 입력한다. 

    ```
    ./elasticsearch
    ```


    정상적으로 설치가 완료되었다면 
    `localhost:9200` 접속 시 아래와 같은 화면을 확인할 수 있다. 
    
    ![elasticsearch install result](elasticsearch-install-result.png)


    ---
    Elasticsearch 설치 방법: <https://www.elastic.co/guide/kr/elasticsearch/reference/current/gs-installation.html>
<br>
<br>

- **Kibana 설치**<br><br>


    아래 링크에서 Kibana 압축 파일을 다운받을 수 있다. 


    파일 다운로드: <https://www.elastic.co/kr/downloads/kibana>
    혹은, 다음과 같은 명령어를 이용해 파일을 다운받는다.

    ```
    wget https://artifacts.elastic.co/downloads/kibana/kibana-7.6.1-linux-x86_64.tar.gz
    ```

    예시에서는 Kibana 7.6.1 버전을 사용했다. <br>(설치된 Elasticsearch와 같은 버전을 사용해야 한다)


    파일 다운이 완료되면, 아래 명령어를 이용해 sha1sum 또는 `shasum`으로 생성된 SHA를 published SHA와 비교하고, 압축을 푼다. 

    ```
    sha1sum kibana-7.6.1-linux-x86_64.tar.gz 
    tar -xzf kibana-7.6.1-linux-x86_64.tar.gz
    ```
    
    
    다음과 같은 명령어를 실행해 Kibana를 시작한다. 

    ```
    cd kibana-7.6.1-linux-x86_64
    ./bin/kibana
    ```

    
    
    정상적으로 설치가 완료되었다면 
    `localhost:5601` 접속 시 아래와 같이 Kibana 시작 화면을 확인할 수 있다.

    ![kibana install result](kibana-install-result.png)
  
  

    아래와 같은 화면이 뜰 경우, elasticsearch와의 버전이 맞지 않거나 elasticsearch가 설치되지 않은 것이다.

    ![kibana install wrong result](kibana-install-wrong-result.png)

    ---
    Kibana 설치 방법: <https://www.elastic.co/guide/kr/kibana/current/targz.html>