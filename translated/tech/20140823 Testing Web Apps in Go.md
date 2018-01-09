# ���� Go ���� Web Ӧ��

![image](http://markjberger.com/assets/img/2014-08-23/cover.jpg)


��������Ͼ��ʱ�䳢������ Go ��дһ����վСӦ�á��� Go ��׼������һЩ�ǳ����İ������� Web Ӧ�ÿ�����ʹ�ò����ҷǳ�ϲ��ʹ�����ǡ�ʵ���ϣ��� Go �ٷ��� wiki ����һ����д web Ӧ�õ�С�̡̳����ǣ�ȴû���ἰ����ñ�׼��ȥ���� web Ӧ�ã�����Ҳû���������ȽϺõ�һ��������


������һ�Ե���̬�����Լ�����Ŀ������һЩ���ԣ��ҷ����˲��� Go ������ Web Ӧ�õĹؼ�������ͨ��ʹ�ø߼��ĺ�����ʵ������ע�롣

## ����ע��

ͨ��[����ע��](https://en.wikipedia.org/wiki/Dependency_injection)����ϣ���ܴﵽ����֧���������еĹ���Ҫ������һ��Ŀ��

Ȼ�����տ�ʼ��̫�����ô����������Ľ̶̳���������дһ�� Web Ӧ�ã�
```
package main

import (
    "fmt"
    "net/http"
)

func handler(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintf(w, "Hi there, I love %s!", r.URL.Path[1:])
}

func main() {
    http.HandleFunc("/", handler)
    http.ListenAndServe(":8080", nil)
}
```
��Դ: [��дһ�� Web Ӧ�� - Golang Wiki](https://golang.org/doc/articles/wiki/)


������˵����һ�����õĴ�����������������Ҫ֪�������һ�����ݿ�ʱ������ʲô�أ�����һ���ⲿ�İ�Ҫ�������ǵ� sessions ���� [Gorilla Session](https://github.com/gorilla/sessions) ����ô��ͨ�������ǻ�ʵ����һ�����ݿ���������߲���һ��ȫ�ֵ� session ���ҽ���Ч�����ó�һ�졣������������ͼ����ʱ�����������鷳������Ǳ�дһ��������Ϊ�㴴����صĲ�����
```
package main

import (
    "fmt"
    "net/http"
    "github.com/markberger/database"
)

type AppDatabase interface {
    GetBacon() string
}

func homeHandler(db AppDatabase) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        fmt.Fprintf(w, "Hi there, I love %s!", db.GetBacon())
    })
}

func main() {
    db := database.NewDatabase()
    http.HandleFunc("/", homeHandler(db))
    http.ListenAndServe(":8080", nil)
}
```

�����Ƿǳ��õģ���Ϊ�������ǾͲ�����Ҫȫ�ֱ����ˡ����������ܷǳ����ɵ� mocking ����һ������Ķ����������Ա���ԵĲ��Է�����������ֻ��Ҫʵ�� `GetBacon` �������,���ܼ򵥵ش���һ�����ݿ� mock ������ӿڵ���������

## ����

����������Ҫһ����Ŀ����ʼ��д���ǵĲ��ԡ��ؼ��ǲ��� `http.Handler` ���õ��� [net/http/httptest](https://golang.org/pkg/net/http/httptest/) ���е� `httptest.ResponseRecorder` ����ķ�����
```
package main

import(
    "net/http"
    "net/http/httptest"
    "testing"
)

type MockDd struct {}

function (db MockDb) GetBacon() {
    return "bacon"
}

function TestHome(t *testing.T) {
    mockDb := MockDb{}
    homeHandle := homeHandler(mockDb)
    req, _ := http.NewRequest("GET", "", nil)
    w := httptest.NewRecorder()
    homeHandle.ServeHTTP(w, req)
    if w.Code != http.StatusOK {
        t.Errorf("Home page didn't return %v", http.StatusOK)
    }
}
```
��Ȼ����Щ�����ܺܺõĸ��ã������ǽ��� POST �������ʱ����ˣ���һֱ��ʹ����Щ���������в��Բ�����

```
package main

import (
    "net/http"
    "net/http/httptest"
    "net/url"
    "testing"
)

type HandleTester func(
    method string,
    params url.Values,
) *httptest.ResponseRecorder

// Given the current test runner and an http.Handler, generate a
// HandleTester which will test its given input against the
// handler.

func GenerateHandleTester(
    t *testing.T,
    handleFunc http.Handler,
) HandleTester {

    // Given a method type ("GET", "POST", etc) and
    // parameters, serve the response against the handler and
    // return the ResponseRecorder.

    return func(
        method string,
        params url.Values,
    ) *httptest.ResponseRecorder {

        req, err := http.NewRequest(
            method,
            "",
            strings.NewReader(params.Encode()),
        )
        if err != nil {
            t.Errorf("%v", err)
        }
        req.Header.Set(
            "Content-Type",
            "application/x-www-form-urlencoded; param=value",
        )
        w := httptest.NewRecorder()
        handleFunc.ServeHTTP(w, req)
        return w
    }
}

function TestHome(t *testing.T) {
    mockDb := MockDb{}
    homeHandle := homeHandler(mockDb)
    test := GenerateHandleTester(t, homeHandle)
    w := test("GET", url.Values{})
    if w.Code != http.StatusOK {
        t.Errorf("Home page didn't return %v", http.StatusOK)
    }
}
```

������ص���ϸ�÷�������[���](https://github.com/markberger/carton/blob/master/api/auth_test.go)�鿴�ҵ�С��Ŀ��

������и��õķ��������ñ�׼����� web Ӧ�ò��ԣ��������ʱ�������Ի���ҷ� email��


----------------

via: http://markjberger.com/testing-web-apps-in-golang/

���ߣ�[ALI M.MIRZAEE](http://alimrz.com/about/)
���ߣ�[zhuCheer](https://github.com/zhuCheer)
У�ԣ�[У����ID](https://github.com/У����ID)

������ [GCTT](https://github.com/studygolang/GCTT) ԭ�����룬[Go ������](https://studygolang.com/) �����Ƴ�