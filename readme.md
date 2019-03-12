Состав проекта:
=======================
 * /src/main/java/com/myteam/helloworld/HelloWorldObject.java - определение типа данных для проекта
 * /src/main/java/com/myteam/helloworld/HelloWorldObjectWithLocale.java - расширенный тип данных
 * /src/main/resources/com/myteam/helloworld/Hello world rule.drl - правила на языке drl для примера 1
 * /src/main/resources/com/myteam/helloworld/Define lang and second world.gdst - правила реализованные через таблицу принятия решения для примера 2
 
 
Инструкция по запуску проекта на KIE WB & KIE server
=======================
1. Импортируем проект в KIE WB: в окне "space" в правом верхнем углу кнопка меню с тремя вертикальными точками, пункт "Import project", указываем путь к этому git repo без логина/пароля.
2. Собираем проект: в окне проекта кнопка Build
3. Деплоим: кнопка Deploy
4. Наш проект должен быть задеплоен в контейнер с именем HelloWorld_1.0.2
5. Формируем REST вызов (basic авторизация):
```text
curl -X POST \
  http://<ИМЯ_СЕРВЕРА>:<ПОРТ>/kie-server/services/rest/server/containers/instances/HelloWorld_1.0.2 \
  -H 'Authorization: Basic ************************' \
  -H 'Content-Type: application/json' \
  -d '{
    "lookup": "HW SESS STATELESS",
    "commands": [
        {
            "insert": {
                "out-identifier": "demo1Object",
                "object": {
                    "com.myteam.helloworld.HelloWorldObject": {
                        "firstStr": "Hello"
                    }
                }
            }
        },
        
        {
            "insert": {
                "out-identifier": "demo2Object",
                "object": {
                    "com.myteam.helloworld.HelloWorldObjectWithLocale": {
                        "firstStr": "Привет"
                    }
                }
            }
        },
        
        {
            "insert": {
                "out-identifier": "demo3Object",
                "object": {
                    "com.myteam.helloworld.HelloWorldObjectWithLocale": {
                        "firstStr": "Привiт"
                    }
                }
            }
        },        
        

        {
            "fire-all-rules": {
                "out-identifier": "countFiredRules"
            }
        }
       
        
    ]
}'

```

В этом примере demo1Object будет обработан правилами из файла /src/main/resources/com/myteam/helloworld/Hello world rule.drl
Остальные 2 из таблицы принятия решения. Результаты будут помещены в те же объекты в поля secondStr, fullStr и originalLang



Ответ будет такой:
```json
{
    "type": "SUCCESS",
    "msg": "Container HelloWorld_1.0.2 successfully called.",
    "result": {
        "execution-results": {
            "results": [
                {
                    "value": {
                        "com.myteam.helloworld.HelloWorldObject": {
                            "firstStr": "Hello",
                            "secondStr": "World",
                            "fullStr": "Hello World"
                        }
                    },
                    "key": "demo1Object"
                },
                {
                    "value": 9,
                    "key": "countFiredRules"
                },
                {
                    "value": {
                        "com.myteam.helloworld.HelloWorldObjectWithLocale": {
                            "firstStr": "Привiт",
                            "secondStr": "Свiт",
                            "fullStr": "Привiт Свiт",
                            "originalLang": "ua"
                        }
                    },
                    "key": "demo3Object"
                },
                {
                    "value": {
                        "com.myteam.helloworld.HelloWorldObjectWithLocale": {
                            "firstStr": "Привет",
                            "secondStr": "мир",
                            "fullStr": "Привет мир",
                            "originalLang": "ru"
                        }
                    },
                    "key": "demo2Object"
                }
            ],
            "facts": [
                {
                    "value": {
                        "org.drools.core.common.DefaultFactHandle": {
                            "external-form": "0:1:797160939:797160939:8:DEFAULT:NON_TRAIT:com.myteam.helloworld.HelloWorldObject"
                        }
                    },
                    "key": "demo1Object"
                },
                {
                    "value": {
                        "org.drools.core.common.DefaultFactHandle": {
                            "external-form": "0:3:307243939:307243939:12:DEFAULT:NON_TRAIT:com.myteam.helloworld.HelloWorldObjectWithLocale"
                        }
                    },
                    "key": "demo3Object"
                },
                {
                    "value": {
                        "org.drools.core.common.DefaultFactHandle": {
                            "external-form": "0:2:1073930568:1073930568:10:DEFAULT:NON_TRAIT:com.myteam.helloworld.HelloWorldObjectWithLocale"
                        }
                    },
                    "key": "demo2Object"
                }
            ]
        }
    }
}
```
