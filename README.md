# 1. 프로젝트 소개
미세먼지 공공 API를 이용해 미세먼지 등급과 다음날 미세먼지 예보를 보여주는 애플리케이션 개발 프로젝트   

## 1.1 프로젝트 요구사항 분석 스프레드시트 
[https://docs.google.com/spreadsheets/d/1tAolHvdSZUqzN1v54wQQhXsosJQsSVjXHvc1UjUMKPA/edit#gid=0](https://docs.google.com/spreadsheets/d/1tAolHvdSZUqzN1v54wQQhXsosJQsSVjXHvc1UjUMKPA/edit#gid=0)

# 2. 서비스 링크
[서비스 링크]()

# 3. 사용기술
- API 개발 : 스프링 프레임워크
- API 배포 : AWS EC2
- Mock API 개발 : PostMan

# 4. 개발내용
- 클라이언트가 현재 위치의 미세먼지 데이터를 요청하면 가까운 측정소 24시간 미세먼지 데이터를 응답하는 API 개발 
- 클라이언트가 내일 예보 이미지를 요청하면 3시간 단위 예보 이미지를 응답하는 API 개발 
- 클라이언트가 내일 예보 개황과 시도별 미세먼지 등급을 요청하면 해당하는 데이터를 응답하는 API 

# 5. 어려움과 해결책
## 5.1 어려움 : 근처 측정소 미세먼지 응답하는 API의 잘못된 설계  
- 문제 : 경도와 위도가 GET방식으로 전달되면 1)해당하는 읍,면,동을 구하고  2) 읍,면,동으로 TM좌표를 얻고, 3) TM좌표로 근처 측정소를 구하고, 4) 근처 측정소의 미세먼지 데이터를 응답하는 것이 요구사항이었다. 하지만 나는 경도와 위도가 전달되면 읍,면,동 이름을 얻어서 측정소 데이터를 구하려고 했다. 이 방법의 문제점은 측정소의 이름이 항상 읍,면,동이 아니라는 것. 그렇기 때문에 TM좌표를 통해 정확하게 근처 측정소 이름을 구해야 한다. 
- 해결책 : 외부 API를 통해 읍,면,동을 구하고 공공API를 통해 TM좌표, 근처 측정소, 미세먼지 데이터를 얻은 뒤 응답했다. 

## 5.2 어려움 : 
- 문제 : AWS EC2를 통해 배포를 완료한 후 클라이언트 요청을 받았는데 `java.net.ConnectException: Connection timed out` 에러가 발생해서 공공API에 요청하지 못하는 상황 
- 해결책 : 공공API를 요청할 때 연결되는 시간이 지연되기 때문에 `urlConnection.setConnectTimeout(1000);` 코드를 추가해 연결 시간을 늘렸다.

# Rest API 문서
## Heroku 배포 URL (heroku9 -> heroku99 변경) 
- 사용자 근처 측정소 데이터 : GET http://dust99.herokuapp.com/location?latitude=37.4624272&longitude=126.97828419999999
- 예보 이미지 : GET http://dust99.herokuapp.com/images
- 시도별 예보 문구 및 등급 : GET http://dust99.herokuapp.com/information

## 실제 배포 URL 
- 사용자 근처 측정소 데이터 : GET http://52.79.74.109:8080/location?latitude=37.4624272&longitude=126.97828419999999
- 예보 이미지 : GET http://52.79.74.109:8080/images
- 시도별 예보 문구 및 등급 : GET http://52.79.74.109:8080/information


## 응답되는 Json 형식 
- 사용자 근처 측정소와 지난 24시간 미세먼지 농도 응답 결과 
```
{
    "stationName": "동작구",
    "dustValues": [
        {
            "pm10Value": "50",
            "datetime": "13",
            "pm10Grade": "2"
        },
        {
            "pm10Value": "41",
            "datetime": "12",
            "pm10Grade": "2"
        },
        {
            "pm10Value": "36",
            "datetime": "11",
            "pm10Grade": "2"
        },
        {
            "pm10Value": "36",
            "datetime": "10",
            "pm10Grade": "2"
        },
        {
            "pm10Value": "35",
            "datetime": "09",
            "pm10Grade": "2"
        },
        {
            "pm10Value": "35",
            "datetime": "08",
            "pm10Grade": "2"
        },
        .....
}
```

- 예보 이미지
```
{
images: [
"http://www.airkorea.or.kr/file/viewImage/?atch_id=139069",
"http://www.airkorea.or.kr/file/viewImage/?atch_id=139070",
"http://www.airkorea.or.kr/file/viewImage/?atch_id=139071"]
}
```

- 예보 문구, 지역별 등급
```
{
  "informOverall": "수도권·강원영서·충청권·호남권·부산·경남·제주권은 ‘나쁨’, 그 밖의 권역은 ‘보통’으로 예상됨. 다만, 그 밖의 권역에서도 ‘나쁨’ 수준의 농도가 일시적으로 나타날 수 있음",
  "informGrade": "서울 : 나쁨,제주 : 나쁨,전남 : 나쁨,전북 : 나쁨,광주 : 나쁨,경남 : 나쁨,경북 : 보통,울산 : 보통,대구 : 보통,부산 : 나쁨,충남 : 나쁨,충북 : 나쁨,세종 : 나쁨,대전 : 나쁨,영동 : 보통,영서 : 나쁨,경기남부 : 나쁨,경기북부 : 나쁨,인천 : 나쁨"
}
```
