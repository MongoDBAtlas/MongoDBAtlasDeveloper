<img src="https://companieslogo.com/img/orig/MDB_BIG-ad812c6c.png?t=1648915248" width="50%" title="Github_Logo"/> <br>


# MongoDB Atlas Training for Developers

### [&rarr; Index 생성 및 테스트](#Index)
### [&rarr; Flask Application](#Application)
### [&rarr; Search in Aggregate](#Aggregate)
### [&rarr; Vector Search](#Vector)

실습은 Python Application을 이용하여 테스트 할 수 있으며 익숙하지 않은 경우 Aggregate를 이용하여 실습 할 수 있습니다. 
<br>

### Index
MongoDB Atlas Search를 이용하여 Data pipeline 구성 없이 Atlas에 저장된 데이터에 대해 full text 검색을 수행 합니다.

#### full text index 생성
Atlas console 에 로그인 후 데이터베이스 클러스터를 선택 후 Search를 클릭 합니다.   
Free tier는 3개의 인덱스 까지 생성 가능 합니다.   
<img src="/02.atlas-search/image/images01.png" width="70%" height="70%">    

인덱스 생성은 Json 으로 직접 입력 하거나 UI를 통해 생성 할 수 있습니다. Visual Editor 를 선택 합니다.
<img src="/02.atlas-search/image/images02.png" width="70%" height="70%">    

컬렉션은 sample_mflix.movies 를 선택 하고 인덱스 이름은 default로 합니다.
<img src="/02.atlas-search/image/images03.png" width="70%" height="70%">  

Analyzer 등은 기본 설정인 standard로 선택 하며 Dynamic을 기본값인 On을 선택 합니다. (전체 기본값으로 하여 줍니다)   
<img src="/02.atlas-search/image/images20.png" width="70%" height="70%">  

Create Search Index를 클릭 하여 인덱스를 생성하여 줍니다. 작업은 백그라운드에서 실행 되며 몇분 후에 완료 됩니다.
<img src="/02.atlas-search/image/images04.png" width="70%" height="70%">    


#### 기본 Full text Search
Atlas Console 의 Aggregation 항목을 선택 하고 검색 관련한 pipeline 을 다음과 같이 생성 하여 줍니다.

<img src="/02.atlas-search/image/images05.png" width="70%" height="70%">    

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

<img src="/02.atlas-search/image/images06.png" width="70%" height="70%">    

쿼리가 실행 되어 결과가 보여 지게 됩니다. 해당 필드에 해당 검색어로 검색한 결과로 보여 집니다.

<img src="/02.atlas-search/image/images07.png" width="70%" height="70%">    

### Application
Python을 설치 한 후 application 폴더에 Flask project를 다운로드 후 config.py 에 MongoDB를 접근하기 위한 Connection String을 입력 합니다.    

`````
mongo_uri="mongodb+srv://altas-account:<password>@cluster0.****.mongodb.net/"
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

애플리케이션에 접속 합니다.  (http://localhost:5010/)   

<img src="/02.atlas-search/image/images08.png" width="80%" height="80%">    


#### 일반 텍스트 검색
Server.py 의 20 라인에 다음을 확인 합니다. 
`````
    with open("queries/query01.json", "r", encoding = 'utf-8') as query_file:
`````

queries 폴더에 query01.json을 이용한 텍스트 검색으로 movies 컬렉션에 title 컬럼에서 갬색을 진행 합니다. 결과는 3개 항목 만을 기준으로 하며 검색 Score를 함께 보여 줍니다.    

다음은 "crime"으로 검색한 결과 입니다. (한단어로 검색 하기)       
<img src="/02.atlas-search/image/images09.png" width="70%" height="70%">    

사용한 Query는 다음과 같습니다. (queries/query01.json)  
path 가 "title" 이며 검색 단어(!!queryParameter!!)를 받아 검색을 진행 합니다.     
`````
[
    {
      "$search": {
        "index": "default",
        "text": {
          "query": "!!queryParameter!!",
          "path": "title"
        }
      }
    },
    {
			"$limit" : 3
		},
		{
      "$project": {
        "_id" : 0,
        "poster" : 1,
        "year" : 1,
        "imdb.rating" : 1,
        "runtime" : 1,
        "fullplot" : 1,
        "title" : 1,
        "cast" : 1,
        "score": { "$meta": "searchScore" }
      }
    }
]
`````


두개 이상의 단어를 입력하여 줄거리에서 검색을 진행 합니다.
진행을 위해 Server.py 의 20 라인을 다음과 같이 변경합니다.
`````
    with open("queries/query02.json", "r", encoding = 'utf-8') as query_file:
`````

fullplot 항목에서 입력한 키워드로 검색한 합니다. (total recall 로 검색한 결과) 결과는 전체 검색 결과를 리턴 합니다.    
스페이스 구분된 두개의 단어("total","recall")가 각각 검색이 되어 나온 것을 확인 할 수 있습니다. (검색 결과가 10개로 total과 recall을 모두 포함하는 영화가 높은 스코어로 상위에 나오지며 하위에 Grindhouse 등 한단어만 포함하는 영화가 나오는 것을 확인 할 수 있습니다.)    
<img src="/02.atlas-search/image/images21.png" width="80%" height="80%">    

사용한 Query는 다음과 같습니다. (queries/query02.json)  
path 가 "title","fullplot","plot" 이며 검색 단어(!!queryParameter!!)를 받아 검색을 진행 합니다.     
`````
[
    {
      "$search": {
        "index": "default",
        "text": {
          "query": "!!queryParameter!!",
          "path": ["title","fullplot","plot"]
        }
      }
    },
    {
			"$limit" : 10
		},
		{
      "$project": {
        "_id" : 0,
        "poster" : 1,
        "year" : 1,
        "imdb.rating" : 1,
        "runtime" : 1,
        "fullplot" : 1,
        "title" : 1,
        "cast" : 1,
        "score": { "$meta": "searchScore" }
      }
    }
]
`````

#### 문장 검색
total recall로 검색을 하면 total과 recall 으로 검색 한 결과가 보여 진것을 확인 하였습니다. 이를 하나의 단어(문장)으로 인식하고 검색을 진행 합니다.   
"total recall"이 포함된 영화를 검색하여 그 결과가 2개만 있는 것을 확인 할 수 있습니다.   
`````
    with open("queries/query06.json", "r", encoding = 'utf-8') as query_file:
`````
<img src="/02.atlas-search/image/images22.png" width="80%" height="80%">    

사용한 Query는 다음과 같습니다. (queries/query06.json)  
path 가 "title","fullplot","plot" 이나 검색이 "phrase"로 검색 단어(!!queryParameter!!)가 포함된 영화를 검색 합니다.     
`````
[
    {
      "$search": {
        "index": "default",
        "phrase": {
          "query": "!!queryParameter!!",
          "path": ["title","fullplot","plot"],
					"slop" : 0
        }
      }
    },
    {
      "$limit" : 10
    },
    {
      "$project": {
        "_id" : 0,
        "poster" : 1,
        "year" : 1,
        "imdb.rating" : 1,
        "runtime" : 1,
        "fullplot" : 1,
        "title" : 1,
        "cast" : 1,
        "score": { "$meta": "searchScore" }
      }
    }
]
`````

#### Fuzzy 검색
두개의 단어 new york 을 검색 하는 경우 관련된 영화를 볼 수 있습니다. 사용자가 오타를 입력한 경우 즉 nrw yprk 로 검색을 하면 아무런 결과가 나오지 않습니다. 오타를 인지 하고 처리 해주기 위해 Query 를 변경합니다.    
   
`````
    with open("queries/query09.json", "r", encoding = 'utf-8') as query_file:
`````
<img src="/02.atlas-search/image/images12.png" width="50%" height="50%">    

사용한 Query는 다음과 같습니다. (queries/query09.json)  
path 가 "title"로 입력한 단어를 검색 하며 fuzzy 설정에 따라 1개의 오타를 허용 하여 검색 합니다.     
`````
[
    {
      "$search": {
        "text": {
          "path": "title",
          "query": "!!queryParameter!!",
          "fuzzy": {
            "maxEdits": 1,
            "maxExpansions": 100
          }
        }
      }
    },
    {
      "$limit" : 10
    },
    {
      "$project": {
        "_id" : 0,
        "poster" : 1,
        "year" : 1,
        "imdb.rating" : 1,
        "runtime" : 1,
        "fullplot" : 1,
        "title" : 1,
        "cast" : 1,
        "score": { "$meta": "searchScore" }
      }
    }
]
`````

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
<img src="/02.atlas-search/image/images13.png" width="50%" height="50%">

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

<img src="/02.atlas-search/image/images14.png" width="50%" height="50%">


### Aggregate
Application을 사용하지 않고 직접 mongosh을 이용하여 Query를 작성하여 실행하는 것입니다. 테스트를 위해 Mongosh을 설치 하거나 Compass을 실행 하여 줍니다.

#### 일반 텍스트 검색

title 컬럼을 대상으로 하여 "crime"을 검색 한 것으로 aggregate pipeline으로 검색이 가능 합니다.   

`````
[primary] sample_mflix> let query="crime"

[primary] sample_mflix> let search = {
...       "$search": {
...         "index": "default",
...         "text": {
...           "query": query,
...           "path": "title"
...         }
...       }
...     }

[primary] sample_mflix> let projection = {
...       "$project": {
...         "_id" : 0,
...         "poster" : 1,
...         "year" : 1,
...         "imdb.rating" : 1,
...         "runtime" : 1,
...         "fullplot" : 1,
...         "title" : 1,
...         "cast" : 1,
...         "score": { "$meta": "searchScore" }
...       }
...     }

[primary] sample_mflix> db.movies.aggregate([search,projection, limit])
[
  {
    runtime: 98,
    cast: [
      'Terence Hill',
      'Bud Spencer',
      'David Huddleston',
      'Luciano Catenacci'
    ],
    poster: 'https://m.media-amazon.com/images/M/MV5BN2Q5MThlZTgtMWMyMi00NDIwLWE1NTMtNzFlOGRmM2QxMGY0XkEyXkFqcGdeQXVyNzc5MjA3OA@@._V1_SY1000_SX677_AL_.jpg',
    title: 'Crime Busters',
    fullplot: "Through an improbable series of events and an impossibly bungled supermarket hold-up, down on their luck con men Matt and Wilbur find themselves working with the Miami police force. As they patrol the streets of the city, their main job becomes trying to break the hold of the city's street gangs, including one group of teens in old movie-gangster style clothes, 
    ...
`````

두개 이상의 단어를 입력하여 제목, 줄거리에서 검색을 진행 합니다.

`````
[primary] sample_mflix> let query="total recall"

[primary] sample_mflix> let search = { "$search": { "index": "default", "text": { "query": query, "path": ["title", "fullplot", "plot"] } } }

[primary] sample_mflix> let projection = { "$project": { "_id": 0, "poster": 1, "year": 1, "imdb.rating": 1, "runtime": 1, "fullplot": 1, "title": 1, "cast": 1, "score": { "$meta": "searchScore" } } }

[primary] sample_mflix> let limit = {"$limit":10}

[primary] sample_mflix> db.movies.aggregate([search,projection, limit])
[
  {
    runtime: 113,
    cast: [
      'Arnold Schwarzenegger',
      'Rachel Ticotin',
      'Sharon Stone',
      'Ronny Cox'
    ],
    poster: 'https://m.media-amazon.com/images/M/MV5BYzU1YmJjMGEtMjY4Yy00MTFlLWE3NTUtNzI3YjkwZTMxZjZmXkEyXkFqcGdeQXVyNDc2NjEyMw@@._V1_SY1000_SX677_AL_.jpg',
    title: 'Total Recall',
    fullplot: "Douglas Quaid is haunted by a recurring dream about a journey to Mars. He hopes to find out more about this dream and buys a holiday at Rekall Inc. where they sell implanted memories. But something goes wrong with the memory implantation and he remembers being a secret agent fighting against the evil Mars administrator Cohaagen. Now the story really begins and it's a rollercoaster ride until the massive end of the movie.",
    year: 1990,
    imdb: { rating: 7.5 },
    score: 9.212348937988281
  },
  {
    runtime: 118,
    cast: [
      'Colin Farrell',
      'Kate Beckinsale',
      'Jessica Biel',
      'Bryan Cranston'
    ],
    poster: 'https://m.media-amazon.com/images/M/MV5BN2ZiMDMzYWItNDllZC00ZmRmLWI1YzktM2M5M2ZmZDg1OGNlXkEyXkFqcGdeQXVyNDQ2MTMzODA@._V1_SY1000_SX677_AL_.jpg',
    title: 'Total Recall',
    year: 2012,
    imdb: { rating: 6.3 },
    score: 9.212348937988281
  },
  {
    fullplot: `Six unemployed steel workers, inspired by the Chippendale's dancers, form a male striptease act. The women cheer them on to go for "the full monty" - total nudity.`,
    imdb: { rating: 7.2 },
    year: 1997,
    title: 'The Full Monty',
    poster: 'https://m.media-amazon.com/images/M/MV5BODg0NWFjMTAtNGZjMC00NmZlLThhZDYtM2MzNTU2NDZiZDVmXkEyXkFqcGdeQXVyMDUyOTUyNQ@@._V1_SY1000_SX677_AL_.jpg',
    cast: [ 'Robert Carlyle', 'Mark Addy', 'William Snape', 'Steve Huison' ],
    runtime: 91,
    score: 7.376830101013184
  },
..
`````

3번째로 검색된 영화 "The Full Monty"를 보면 줄거리 부분에 "total nudity"로 인해 total이 검색된 것으로 2개 단어 중 하나만 검색되어도 결과로 나오는 것을 확인 할 수 있습니다. 


#### 문장 검색
total recall로 검색을 하면 total과 recall 으로 검색 한 결과가 보여 진것을 확인 하였습니다. 이를 하나의 단어(문장)으로 인식하고 검색을 진행 합니다.   
"total recall"이 포함된 영화를 검색하여 그 결과가 2개만 있는 것을 확인 할 수 있습니다.   
`````
[primary] sample_mflix> let query="total recall"

[primary] sample_mflix> let search = { "$search": { "index": "default", "phrase": { "query": query, "path": ["title", "fullplot", "plot"], "slop": 0 } } }

[primary] sample_mflix> let projection = { "$project": { "_id": 0, "poster": 1, "year": 1, "imdb.rating": 1, "runtime": 1, "fullplot": 1, "title": 1, "cast": 1, "score": { "$meta": "searchScore" } } }

[primary] sample_mflix> let limit = {"$limit":10}

[primary] sample_mflix> db.movies.aggregate([search,projection, limit])
[
  {
    runtime: 113,
    cast: [
      'Arnold Schwarzenegger',
      'Rachel Ticotin',
      'Sharon Stone',
      'Ronny Cox'
    ],
    poster: 'https://m.media-amazon.com/images/M/MV5BYzU1YmJjMGEtMjY4Yy00MTFlLWE3NTUtNzI3YjkwZTMxZjZmXkEyXkFqcGdeQXVyNDc2NjEyMw@@._V1_SY1000_SX677_AL_.jpg',
    title: 'Total Recall',
    fullplot: "Douglas Quaid is haunted by a recurring dream about a journey to Mars. He hopes to find out more about this dream and buys a holiday at Rekall Inc. where they sell implanted memories. But something goes wrong with the memory implantation and he remembers being a secret agent fighting against the evil Mars administrator Cohaagen. Now the story really begins and it's a rollercoaster ride until the massive end of the movie.",
    year: 1990,
    imdb: { rating: 7.5 },
    score: 9.212348937988281
  },
  {
    runtime: 118,
    cast: [
      'Colin Farrell',
      'Kate Beckinsale',
      'Jessica Biel',
      'Bryan Cranston'
    ],
    poster: 'https://m.media-amazon.com/images/M/MV5BN2ZiMDMzYWItNDllZC00ZmRmLWI1YzktM2M5M2ZmZDg1OGNlXkEyXkFqcGdeQXVyNDQ2MTMzODA@._V1_SY1000_SX677_AL_.jpg',
    title: 'Total Recall',
    year: 2012,
    imdb: { rating: 6.3 },
    score: 9.212348937988281
  }
]
`````

#### Fuzzy 검색
두개의 단어 new york 을 검색 하는 경우 관련된 영화를 볼 수 있습니다. 사용자가 오타를 입력한 경우 즉 nrw yprk 로 검색을 하면 아무런 결과가 나오지 않습니다. 오타를 인지 하고 처리 해주기 위해 Query 를 변경합니다.    
   
`````
[primary] sample_mflix> let query="naw yark"

[primary] sample_mflix> let search = { "$search": { "text": { "path": "title", "query": query, "fuzzy": { "maxEdits": 1, "maxExpansions": 100 } } } }

[primary] sample_mflix> let projection = { "$project": { "_id": 0, "poster": 1, "year": 1, "imdb.rating": 1, "runtime": 1, "fullplot": 1, "title": 1, "cast": 1, "score": { "$meta": "searchScore" } } }

[primary] sample_mflix> let limit = {"$limit":10}

Atlas atlas-txdmjn-shard-0 [primary] sample_mflix> db.movies.aggregate([search,projection, limit])
[
  {
    runtime: 155,
    cast: [
      'Liza Minnelli',
      'Robert De Niro',
      'Lionel Stander',
      'Barry Primus'
    ],
    poster: 'https://m.media-amazon.com/images/M/MV5BYzZjM2RlZWMtZjVhNC00ODAyLTg0MDEtZDNmNjU4ODg2YjY3XkEyXkFqcGdeQXVyNjc1NTYyMjg@._V1_SY1000_SX677_AL_.jpg',
    title: 'New York, New York',
    fullplot: 'The day WWII ends, Jimmy, a selfish and smooth-talking musician, meets Francine, a lounge singer. From that moment on, their relationship grows into love as they struggle with their careers and aim for the top.',
    year: 1977,
    imdb: { rating: 6.7 },
    score: 4.38106107711792
  },
  {
    runtime: 153,
    cast: [
      'John Abraham',
      'Neil Nitin Mukesh',
      'Katrina Kaif',
      'Irrfan Khan'
    ],
    poster: 'https://m.media-amazon.com/images/M/MV5BYTljMjMyYjEtZjRlNi00MGE5LTk5MDQtYzAyYTc3ODJiN2M2XkEyXkFqcGdeQXVyNTkzNDQ4ODc@._V1_SY1000_SX677_AL_.jpg',
    title: 'New York',
    fullplot: "After being apprehended, detained, humiliated, and denied legal counsel by the Federal Bureau of Investigation, Omar Ehzaz, originally from Delhi's Lajpatnagar, relates to the investigator, Roshan, assigned to his case, how he arrived in New York during 1999; his friendship with Samir Shaikh and Student Counselor, Maya; the events of September 11, 2001; the subsequent paranoia, fear, racial profiling, generated and aggravated by the tyrannical right-winged regime of George W. Bush, and how he came to be in possession of several Ak47s and plastic explosives that were confiscated from his taxi-cab.",
    year: 2009,
    imdb: { rating: 6.7 },
    score: 4.040346145629883
  },

`````
path 가 "title"로 입력한 단어를 검색 하며 fuzzy 설정에 따라 1개의 오타를 허용 하여 검색 합니다.     


### Vector
영화에 대해 단어를 이용한 검색이 아닌 문장으로 의미를 작성하여 검색을 진행 합니다. embedded_movies 컬렉션에 사전에 OpenAI 의 "ada-002-text"를 이용하여 vector data를 생성 한 것입니다.

sample_mflix.embedded_movies 컬렉션의 데이터로 plot_embedding 필드에 "ada-002-text" 서비스에 의해 plot 정보가 vector로 계산되어져 저장된 것입니다.    

<img src="/02.atlas-search/image/images30.png" width="50%" height="50%">

해당 필드에 인덱스를 생성 하여 줍니다. Vector 의 차원은 1536이며 비교 연산은 euclidean을 사용 합니다.  
Atlas console에서 인덱스를 생성 하여 줍니다. 인덱스는 UI를 통해서 생성이 지원되지 않음으로 JSON을 이용한 생성을 선택 합니다.   


인덱스 이름을 vector_index로 지정 하고 plot_embedding 필드에 인덱스를 생성하며 type은 knnVector로 지정하고 차원 정보는 1536, 유사도는 euclidean을 입력 하여 줍니다. 장르 정보를 지정하여 검색 하기 위해 genres도 추가 하여 줍니다.     

`````
{
  "mappings": {
    "dynamic": true,
    "fields": {
      "plot_embedding": {
        "type": "knnVector",
        "dimensions": 1536,
        "similarity": "euclidean"
      },
      "genres": {
        "type": "token",
        "normalizer": "lowercase"
      }
    }
  }
}
`````

인덱스 생성 까지는 1-2분 정도가 소요되며 생성 완료는 search index 페이지에서 확인 가능 합니다.    

<img src="/02.atlas-search/image/images31.png" width="70%" height="70%">


검색을 위해서 openAI (https://openai.com/)에 무료 회원 가입 후 회원 정보 페이지에서 API 사용을 위한 API key를 생성 합니다.   


<img src="/02.atlas-search/image/images32.png" width="70%" height="70%">

생성된 API 키는 다시 조회가 불가능 함으로 메모 하여 둡니다. 

OpenAI API를 이용하여 검색할 문장의 Vector 데이터를 생성하여 줍니다.   
다음과 같이 Authorization에 Bearer로 생성한 API key를 입력 하여 줍니다. Body의 data 에 검색할 문장을 input에 넣어 주고 모델을 text-embedding-ada-002로 입력 하여 줍니다. (해당 모델로 input 데이터에 대한 vector 값을 생성 하여 줍니다.)

`````
curl --location 'https://api.openai.com/v1/embeddings' \
--header 'Content-Type: application/json' \
--header 'Authorization: Bearer sk-bPmFRvprJt1VWs****' \
--data '{
    "input": "Experience of shipwreck by Typhoon and surviving alone in small island.",     
    "model": "text-embedding-ada-002"
  }'
`````

호출의 결과로 Vector 데이터가 리턴이 됩니다. 그 결과는 다음과 같습니다.    

`````
{
    "object": "list",
    "data": [
        {
            "object": "embedding",
            "index": 0,
            "embedding": [
                -0.0028920018,
                -0.027676977,
                0.007235899,
                ...
                0.00018032902,
                -0.026289087
            ]
        }
    ],
    "model": "text-embedding-ada-002-v2",
    "usage": {
        "prompt_tokens": 16,
        "total_tokens": 16
    }
}
`````

embedding 부분을 복사 하여 Query에 넣어주면 검색이 가능 합니다.
다음은 Vector를 이용한 검색 Query입니다.   
`````
[primary] sample_mflix> let vector = [
                -0.0028920018,
                -0.027676977,
                0.007235899,
                ...
                0.00018032902,
                -0.026289087
            ]

[primary] sample_mflix> db.embedded_movies.aggregate([ { "$vectorSearch": { "index": "vector_index", "path": "plot_embedding", "queryVector": vector, "numCandidates": 200, "limit": 10 } }, { "$project": { "_id": 0, "title": 1, "genres": 1, "plot": 1, "released": 1, "score": { $meta: "vectorSearchScore" } } }] )
[
  {
    plot: 'A shipwrecked survivor discovers a remote island with a mad scientist.',
    genres: [ 'Adventure', 'Fantasy', 'Horror' ],
    title: 'The Island of Dr. Moreau',
    released: ISODate("1977-07-13T00:00:00.000Z"),
    score: 0.7824490666389465
  },
  {
    plot: 'After a collision with a shipping container at sea, a resourceful sailor finds himself, despite all efforts to the contrary, staring his mortality in the face.',
    genres: [ 'Action', 'Adventure', 'Drama' ],
    title: 'All Is Lost',
    released: ISODate("2013-11-07T00:00:00.000Z"),
    score: 0.7801877856254578
  },
  {
    plot: 'A terrifying tale of survival in the mangrove swamps of Northern Australia',
    genres: [ 'Action', 'Drama', 'Horror' ],
    title: 'Black Water',
    released: ISODate("2008-04-24T00:00:00.000Z"),
    score: 0.7797785401344299
  },
  {
    plot: 'A young man who survives a disaster at sea is hurtled into an epic journey of adventure and discovery. While cast away, he forms an unexpected connection with another survivor: a fearsome Bengal tiger.',
    genres: [ 'Adventure', 'Drama', 'Fantasy' ],
    title: 'Life of Pi',
    released: ISODate("2012-11-21T00:00:00.000Z"),
    score: 0.7781470417976379
  },
  {
    plot: "This is a survival story - a Hemingway's 'Old Man and the Sea' as if written for our days.",
    genres: [ 'Action', 'Adventure', 'Drama' ],
    title: 'The Old Man',
    released: ISODate("2014-09-08T00:00:00.000Z"),
    score: 0.7648832201957703
  },
  {
    plot: "This is a survival story - a Hemingway's 'Old Man and the Sea' as if written for our days.",
    genres: [ 'Action', 'Adventure', 'Drama' ],
    title: 'The Old Man',
    released: ISODate("2014-09-08T00:00:00.000Z"),
    score: 0.7648720741271973
  },
  {
    plot: 'A group of passengers struggle to survive and escape when their ocean liner completely capsizes at sea.',
    genres: [ 'Action', 'Adventure', 'Drama' ],
    title: 'The Poseidon Adventure',
    released: ISODate("1972-12-13T00:00:00.000Z"),
    score: 0.7635738849639893
  },
  {
    plot: 'An unusually intense storm pattern catches some commercial fishermen unaware and puts them in mortal danger.',
    genres: [ 'Action', 'Adventure', 'Drama' ],
    title: 'The Perfect Storm',
    released: ISODate("2000-06-30T00:00:00.000Z"),
    score: 0.7591977119445801
  },
  {
    plot: 'Shipwreck survivors are found on Beiru Island (Infanto tè), which was previously used for atomic tests. The interior is amazingly free of radiation effects, and they believe that they were ...',
    genres: [ 'Fantasy', 'Sci-Fi', 'Thriller' ],
    title: 'Mothra',
    released: ISODate("1962-05-10T00:00:00.000Z"),
    score: 0.7551210522651672
  },
  {
    plot: 'A boating accident runs a young man and woman ashore in a decrepit Spanish fishing town which they discover is in the grips of an ancient sea god and its monstrous half human offspring.',
    genres: [ 'Fantasy', 'Horror', 'Mystery' ],
    title: 'Dagon',
    released: ISODate("2001-10-31T00:00:00.000Z"),
    score: 0.7495789527893066
  }
]

`````

OpenAI의 API 호출 시 원한는 값을 Input에 넣어 Vector 값을 구해서 결과 검색이 가능합니다.   
OpenAI이 계정이 없는 경우 openAI.txt 파일에 전체 값을 넣었으니 활용하여 검색이 가능 합니다.  