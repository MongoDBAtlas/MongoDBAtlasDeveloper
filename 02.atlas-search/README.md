<img src="https://companieslogo.com/img/orig/MDB_BIG-ad812c6c.png?t=1648915248" width="50%" title="Github_Logo"/> <br>


# MongoDB Atlas Training for Developers

### [&rarr; Full Text Search](#Search)

<br>

### Search
MongoDB atlas Search를 이용하여 Data pipeline 구성 없이 Atlas에 저장된 데이터에 대해 full text 검색을 수행 합니다.

#### full text index 생성
Atlas console 에 로그인 후 데이터베이스 클러스터를 선택 후 Search를 클릭 합니다.   
Free tier는 3개의 인덱스 까지 생성 가능 합니다.   
<img src="/02.atlas-search/images/images01.png" width="70%" height="70%">    

인덱스 생성은 Json 으로 직접 입력 하거나 UI를 통해 생성 할 수 있습니다.  Visual Editor 를 선택 합니다.
<img src="/02.atlas-search/images/images02.png" width="70%" height="70%">    

컬렉션은 sample_mflix.movies 를 선택 하고 인덱스 이름은 default로 합니다.
<img src="/02.atlas-search/images/images03.png" width="70%" height="70%">    

인덱스 생성은 백그라운드에서 실행 되며 몇분후에 완료 됩니다.
<img src="/02.atlas-search/images/images04.png" width="70%" height="70%">    


#### 기본 Full text Search
Atlas Console 의 Aggregation 항목을 선택 하고 검색 관련한 pipeline 을 다음과 같이 생성 하여 줍니다.

<img src="/02.atlas-search/images/images05.png" width="70%" height="70%">    

검색을 위한 파이프라인을 구성 합니다. fullplot이라는 컬럼을 대상으로 crime을 검색 합니다.
`````
[
    {
      $search: {
        index: "default",
        text: {
          query: "crime",
          path: "title",
        }
      }
    }
]
`````

<img src="/02.atlas-search/images/images06.png" width="70%" height="70%">    

쿼리가 실행 되어 결과가 보여 지게 됩니다. 해당 필드에 해당 검색어로 검색한 결과로 보여 집니다.

<img src="/02.atlas-search/images/images07.png" width="70%" height="70%">    


#### Flask Application
Python Flask project 를 다운로드 후 config.py 에 MongoDB를 접근하기 위한 Connection String을 입력 합니다.    

`````
mongo_uri="mongodb+srv://altas-account:<password>@cluster0.****.mongodb.net/myFirstDatabase"
`````

필요한 패키를 설치 합니다.
`````
$ pip3 install 'pymongo[srv]'
$ pip3 install pymongo
$ pip3 install flask
`````

애플리케이션을 시작 합니다.
`````
 $ python3 server.py                     
 * Serving Flask app 'server' (lazy loading)
 * Environment: production
   WARNING: This is a development server. Do not use it in a production deployment.
   Use a production WSGI server instead.
 * Debug mode: on
 * Running on http://localhost:5010 (Press CTRL+C to quit)
 * Restarting with stat
 * Debugger is active!
 * Debugger PIN: 220-603-097

`````

애플리케이션에 접속 합니다.    
<img src="/02.atlas-search/images/images08.png" width="70%" height="70%">    


#### 일반 텍스트 검색
Server.py 의 20 라인에 다음을 확인 합니다. 
`````
    with open("queries/query01.json", "r", encoding = 'utf-8') as query_file:
`````

queries 폴더에 query01.json을 이용한 텍스트 검색으로 movies 컬렉션에 title 컬럼에서 갬색을 진행 합니다. 결과는 3개 항목 만을 기준으로 하며 검색 Score를 함께 보여 줍니다.    

다음은 crime으로 검색한 결과 입니다.     
<img src="/02.atlas-search/images/images09.png" width="70%" height="70%">    

두개 이상의 단어를 입력하여 검색을 하기 위해 Server.py 의 20 라인을 다음과 같이 변경합니다.
`````
    with open("queries/query02.json", "r", encoding = 'utf-8') as query_file:
`````

fullplot 항목에서 입력한 키워드로 검색한 합니다. (total recall 로 검색한 결과) 결과는 전체 검색 결괄르 리턴 합니다.     
<img src="/02.atlas-search/images/images10.png" width="70%" height="70%">    


#### 문장 검색
Jimmie Shannon으로 검색을 하면 Jimmie 와 Shannon 으로 검색 한 결과가 보여 지게 됩니다. 이를 한 단어로 하여 검색 합니다.    
`````
    with open("queries/query06.json", "r", encoding = 'utf-8') as query_file:
`````
<img src="/02.atlas-search/images/images11.png" width="50%" height="50%">    

#### Fuzzy 검색
두개의 단어 new york 을 검색 하는 경우 관련된 영화를 볼 수 있습니다. 사용자가 오타를 입력한 경우 즉 nrw yprk 로 검색을 하면 아무런 결과가 나오지 않습니다. 오타를 인지 하고 처리 해주기 위해 Query 를 변경합니다.    
   
`````
    with open("queries/query09.json", "r", encoding = 'utf-8') as query_file:
`````
<img src="/02.atlas-search/images/images12.png" width="50%" height="50%">    


#### 검색어 완성
영화 제목 검색시 자동으로 관련된 단어를 보여 줍니다. 검색 키 입력된 내용에 따라 자동으로 해당 단어로 시작하는 단어를 보여 주게 됩니다. 이를 위해 인덱스를 변경 해 줍니다.  다음 인덱스를 생성 하여 줍니다. 인덱스 이름은 title_autocomplete 로 하여 줍니다.    
   
`````
{
  "mappings": {
    "dynamic": false,
    "fields": {
      "title": [
        {
          "type": "autocomplete",
          "tokenization": "edgeGram",
          "minGrams": 3,
          "maxGrams": 7,
          "foldDiacritics": false
        }
      ]
    }
  }
}
`````
<img src="/02.atlas-search/images/images13.png" width="50%" height="50%">

index.html 파일에 다음 내용을 수정 하여 줍니다. (212 라인) 기존 function() {} 을 findMovieTitles() 로 변경 합니다.   

`````
$('#custom-search-input .typeahead').typeahead({
        hint: true,
        highlight: true,
        minLength: 3
    },
    {
        source: findMovieTitles ()
    });
`````
검색어로 scar 를 입력 하면 scar를 포함한 추천 검색어가 보여 집니다.

<img src="/02.atlas-search/images/images14.png" width="50%" height="50%">