## ��SQL��ΪAPI

����㲻����ʯͷ��ס�ţ���ô��ҲӦ�������������һ���µĶԡ�������Ϊ���񡱵���⡣�ڿ�Դ������Alex Ellis �� [OpenFaas](https://github.com/openfaas/faas)  ��Ŀ�ܵ��˺ܸߵĹ�ע������ [����ѷLambda������Go���Ե�֧��](https://aws.amazon.com/blogs/compute/announcing-go-support-for-aws-lambda/)����Щϵͳ�����㰴�����ݣ�����ͨ�� API ���õķ�ʽ��������� CLI ����

**Lambda/Faas ����Ķ���**

��������ô������ - �������޷��������˶�����ƽ̨������  AWS�����г�Ӫ����Ϊ���������㽫��һ�������������ƽ������ǣ����磬��������£�������ϵͳ��һС���֡��ھ���������ϣ�����ζ�� AWS ���������ƵĽ������ �йܵ����Ӧ�ã��������Ӧ�ã����������ǵ����������ϸ�����Ҫ�Զ�ά������Ӳ����ģ��

���ǣ���������֪����Щ�ˡ�

���ǣ����Ƿ�֪����������������� CGI �д��ڣ��ο�ά���ٿƣ�1993�������� 1997 ��ʽ�� RFC �淶�������Ķ��塣���оɵĶ����־��������ˡ� CGI(Common Gateway Interface) ��Ŀ����:

> �ڼ����ϣ�ͨ�����ؽӿ� (CGI)  Ϊ��վ�������ṩһ����׼��Э�飬�����ڶ�̬��ҳ����������ִ�п���̨Ӧ�ã�Ҳ��Ϊ�����нӿڳ���һ��ִ��Ӧ�ó���
> 
> ��Դ: [ά���ٿ�](https://en.wikipedia.org/wiki/Common_Gateway_Interface)
> 

�� Go ������˵����򵥵� Fass/CGI �������ֻ��Ҫ 10 �����ҵĴ������ʵ�֡�Go ���Եı�׼�����Ѿ�������  [net/http/cgi](https://golang.org/pkg/net/http/cgi/) ��������е����ѹ�����ʵ��һ�� PHP CGI ֻ��Ҫ�����������У�

```
func phpHandler() func(w http.ResponseWriter, r *http.Request) {
    return func(w http.ResponseWriter, r *http.Request) {
        handler := new(cgi.Handler)
        handler.Dir = "/var/www/"
        handler.Path = handler.Dir + "api"
        args := []string{r.RequestURI}
        handler.Args = append(handler.Args, args...)
        fmt.Printf("%v", handler.Args)
        handler.ServeHTTP(w, r)
    }
}
```
�������Ҳ�Ƿǳ��򵥵ģ�
```
http.HandleFunc("/api/", phpHandler())
```
��Ȼ���Ҳ����Ϊʲô���ʹ��������Ϊ����ʱ����ǰ��ʱ�̣� CGI  �������������⣬������������������ϵͳ�ϲ�����ѹ��������ÿ�����󣬾ͻᷢ��һ�� 
```os.Exec``` ���ã�������Բ���һ��ϵͳ�Ѻõĵ��á�ʵ���ϣ�����������κ�����ʵʱ��ͨ�����ķ�����ܿ���ϣ�������ĵ������� 0 .

�����Ϊʲô CGI ����Ϊ�� FastCGI��

> FastCGI ��ͨ�����ؽӿ� ��CGI�� ���ڵ�һ�ֱ仯�� FastCGI ����ҪĿ���Ǽ�����ҳ�������� CGI ����֮��Ĺ��ȵ���ϵ���÷�������һ���Դ���������ҳ����
> 
> ��Դ��[ά���ٿ�](https://en.wikipedia.org/wiki/FastCGI)
 
 �����ڲ���ʵ��һ�� FastCGI �����ڱ�׼����Ҳ��һ��  [net/http/fcgi](https://golang.org/pkg/net/http/fcgi/)�� ,������Ҫ��ʾ����ʵ�ֻ�������������塣��Ȼ������ AWS ��������ĳ���ʱ������ܲ���ô�����������Ϊ����������������ķ�����������Ӳ����
 
 
**CGI �Ľ���취**
 
 ����⼸������ѧ���㶫���Ļ�����һ���Ǵ�����ķ��������������ġ�����ζ�ţ�һ����ĳ�����ݿⱣ����������Ҫ�����ݡ�����һ�� [Quora�ϵĻش�](https://www.quora.com/Which-database-system-s-does-Twitter-use) �� Twitter ʹ��������8�ֲ�ͬ���͵����ݿ⣬�� MySQL,Cassandra �� Redis�������������ӵ����ݿ⡣
 
 ʵ���ϣ��ҵĴ󲿷ֹ�������������ģ������ݿ��ֶ�ȡԱ�����ݣ�Ȼ�������� json ��ʽ����ͨ�� REST �����ṩ��ȥ����Щ��ѯͨ�����ܽ�����һ�� SQL �����ʵ�֣���ȻҲ�кܶ������Ҳ�ǿ��Եġ���ô����������дһЩ������ ``` os.Exec ``` ���óɱ���  SQL �ű���ʵ��һЩ���ܣ��������� CGI ������ʵ�����ǣ�
 
 ��ս�����ˡ�
 
**��SQL��ΪAPI**
 
 �Ҳ��������������Ȼ�����Ȼ �� SQL ��Ϊ API ����������Ϊ�ģ�����ȷʵ��Ҫʵ��һ��Զ�̿��õİ汾����ϣ��ͨ���ڴ����ϴ���һ�� .sql  �ļ���ʵ�� API ���ã������һ�ϣ����� API ���Ե��� http �����е��������������ζ�����ǿ���ͨ�����ݸ� API �Ĳ��������˽��������ѡ���� MySQL �� sqlx ��ʵ���������
 
 �����Ϊ Twitch��Slack��Yotube��Discord д��һЩ��������ˣ��������ܿ���Ҫдһ�� Teleganm �İ汾�ˡ���ʵ���ǵ�Ŀ���ǶԸ���ͨ���������Ƶ����ӣ���¼��Ϣ������һЩͳ����Ϣ���Ҷ�һЩ�������������з����������� Vue.js д�ɵ�ǰ����վ��������Ҫͨ�� API �������Ǵ���һЩ���ݡ���Ȼ�������е� API ������ SQL ��ʵ�֣��������кܴ�һ�����ǿ��Եġ����磺
1. �г����е�ͨ��
1. ͨ��ͨ��ID�г�ͨ��
 
 ���������������˵�����Ʋ�������ʵ�֣����ر𴴽��������ļ������ṩ��Щ��Ϣ��
 ``` api/channels.sql ```(��Ӧ�� ```/api/channels```)
 ```
 select * from channels
 ```
  ``` api/channelByID.sql ```(��Ӧ�� ```/api/channelByID?id=...```)
 ```
select * from channels where id=:id
 ```
 �����㿴������������ SQL ��ѯ��ʵ��һ���µ� API ������Ҫ̫��Ĺ������ҳ������һ��ϵͳ������һ���㴴���� ``` api/name.sql ``` ���ϾͿ���ͨ�� ``` api/name ```���ʵ������� Http ����Ĳ�������װ�� ```map[string]interface{}``` �У�������Ϊ�󶨱������ݸ� SQL ��ѯ��䡣SQL ����������������Щ������
 ��Ҳ����˴�����Ϣ�ĸ�ʽ���������û�����ӵ�������ݿ⣬����һ�� API ��Ӧ�� .sql  �ļ������ڣ�����һ�����µĴ�����Ϣ���أ�
 ```
 {
    "error": {
        "message": "open api/messages.sql: no such file or directory"
    }
}
```
�� SQL ��ѯ��ʹ�� URL ����

�� go �����л������Ĳ�����ֻ��Ҫ����������е� ``` *url.URL```�ṹ���ϵ��� Query() �������ɡ������������ ``` url.Values ``` ��������һ�� ```map[string][]string ```���͵ı�����
������Ҫת��������󲢴��ݵ� sqlx ������С�������Ҫ����һ�� ``` map[string]interface{}``` ����Ϊ������Ҫ���õ� sqlx �����ڲ�ѯʱ�������ָ�ʽ�Ĳ�����[sqlx.NamedStmt.Queryx](https://godoc.org/github.com/jmoiron/sqlx#NamedStmt.Queryx)����������ת�����ǲ��ҷ����ѯ��
```
params := make(map[string]interface{})
urlQuery := r.URL.Query()
for name, param := range urlQuery {
        params[name] = param[0]
}

stmt, err := db.PrepareNamed(string(query))
if err != nil {
        return err
}
rows, err := stmt.Queryx(params)
if err != nil {
        return err
}
```
���ǻ�û�д������ ```rows``` ���������ǿ��Ա������Ի��ÿһ���е���Ϣ��������Ҫ�����Ǽ��뵽һ����Ƭ�У������� API ��������а����Ƿ�װ�� JSON ���档
```
for rows.Next() {
        row := make(map[string]interface{})
        err = rows.MapScan(row)
        if err != nil {
                return err
        }
```
�������������Ȥ�ĵط���ÿһ���а�����ֵ����Ҫת���� JSON �����������Ķ�����
��Ϊ�ײ�������� ```[]uint8```������������Ҫ������ת�����ַ�����������ǲ���ô�������ֽṹ�� JSON ���Զ�ʹ�� base64 ���롣��Ȼ��ѯ�ķ��������� ```map[string]string```����ʾ������ ```uint8``` �� ```byte``` ���͵ı���������ѡ��ʹ������ת����
```
rowStrings := make(map[string]string)
for name, val := range row {
        switch tval := val.(type) {
        case []uint8:
                ba := make([]byte, len(tval))
                for i, v := range tval {
                        ba[i] = byte(v)
                }
                rowStrings[name] = string(ba)
        default:
                return fmt.Errorf("Unknown column type %s %#v", name, spew.Sdump(val))
        }
}
```
����������һ�� ```rowStrings```  �����ʾÿ�����ص� SQL �У����������ɵı��뵽 JSON �С�������Ҫ���ľ��ǰ�������ӵ�һ�����ؽ���У��������벢�ҷ��ر�����ֵ����������Զ�С���Ĵ�������� [titpetric/sql-as-a-service](https://github.com/titpetric/sql-as-an-api) �����ȡ��


**ʹ����֪**

��Ȼ���ַ��������ݿ���Ϊ API ���ʵ���ж��صĺô�������Ϊ�������ʺϱ����㷺�ĵ��ã���������û�������Ҫ���ǡ����磺

**�Խ��������**

���ַ�����ʵ���ܽ���������Ϊ����û����һ����ѯ�����󶨵� ```order by```  �����У���Ϊ SQL ��������ô�������ݵ���ϴҲ����ȫ�����ܵģ��������������ú�������Ϊһ����������������Ĵ�������ȫ�Ĳ����еģ�```ORDER BY :column IF(:order = 'asc', 'asc', 'desc')```��

**����**

Ϊ�˴���ĳ�ַ�ҳ���� MySQL �ṩ�� ```LIMIT $offset, $length``` �Ӿ䡣��Ȼ����Խ���Щ��Ϊ��ѯ��������������������û�������ǣ������ҵ�һ�ִ�������ֵ�İ취�����ǳ������Ľ���ǵõ����������Ĵ��󷵻���Ϣ����δ����ı���...��

**����SQL��ѯ**

ͨ����˵�����ǻ�ִ�ж��� SQL ��������ص�һ��һ���������Ȼ������Ҫ�������ݿ��ϱ�����Ϊʹ��״̬�����������ͨ���Ǳ���ֹ�ģ�����һ����Ҫ��ԭ����Ϊ�˷�ֹ SQL ע�빥������һ������������������������Ӧ���ǿ���ִ�еģ�
```
set @start=:start;
set @length=:length;
select * from channels order by id desc limit @start, @length;
```
��ϧ����ʵ�����в�ͨ����������������û��Զ��ִ�С��������һ�� MySQL �ͻ����г���ִ����Щ��䣬����������Щ�����������ˣ��������ǼȲ����� order by �Ӿ�Ҳ������ limit �Ӿ���ʹ�á�

��ô����û����ҳ��ô��

��һ���Ҽ���������ֵ����߾ȹ��İ취�� Twitch �� Tumblr �� API ��һ���ر�����ԣ����������㴫��һ�� ```since``` ���� ```previousID``` ��ֵ�����ǿ������������� SQL �������ã�
```
select * from messages where id < :previousID order by id desc limit 0, 20
```
��������԰���Ԥ�ȶ��õķ�ҳ��С������һ�������ַ���Ҫ����������и������������������ sonyflake / snowflake �㷨������ID������������һ�������������˳���С�

**����**

SQL ���ݿⲢ���Ǵ�����Ұ�ޣ�ʵ�������Ƿǳ�ǿ��ǿ���Դͷ֮һ���ǿ��Դ������������ߣ����̡�����������ʵ��һ�����ӵ�ҵ���߼����Դ�ͳ�� DBA �������ݿ����Ա����˵����ȡ�����Լ�����һ�� SQL �������Ǻ����׵ġ�
����һ���ܺõĽ���ڿͻ����޷�һ��ִ�ж��� SQL ������Ƶİ취��������Ҫ��������е�ҵ���߼������ݿ���ʵ�֡��������֪�Ĵ󲿷ֳ���Ա��˵�����ǳ������������������¶���������Ҫ��һ��ѵ� IF ���򽻵���

**����**

����������һЩ�򵥵Ĳ�ѯ��ͨ�����Ǿ��ܻ������Ҫ�Ľ���Ļ�����ô����Դ������� SQL  ��Ϊ API �ķ����д�����棬���˽�Լϵͳ�����⣬�㻹�����ṩ����Щ����Ϥ Go ���Ե�����Ϥ SQL  �ĳ���Ա��һ������ϵͳ���ܵķ�����

����Ҫ��һ�� API �����ض����������������ӣ�������Ҫһ�ֽű����ԣ����ܴﵽ������Խ  [PL/SQL](https://en.wikipedia.org/wiki/PL/SQL) ���������ƣ������ڲ�ͬ��ϵ�����ݿ����в�ͬ��ʵ�֡�
���ߣ��������Ӧ����һֱ�ҽ�һ�� JavaScript ���������ͬ [dop251/goja ](https://github.com/dop251/goja)  ������Ȼ���������Ŷӵ�ǰ̨����Ա��һ����/��������Զ�������ǵĳ��ԡ�����Ҳ���ô��� go ����ʵ�ֵ� LUA  ������������Ҫĳ�ֱ����� ES5.1 С�ġ�����ʱ����

**�������������������...**
���������һ���ҵ��飬�Ǿ�̫���ˣ�
- [API Foundations in Go](https://leanpub.com/api-foundations)
- [12 Factor Apps with Docker and Go](https://leanpub.com/12fa-docker-golang)
- [The SaaS Handbook (������)](https://leanpub.com/saas-handbook)

�ұ�֤����Դ���ѧ�����ࡣ��һ������֧����д�������Ƶ����¡����㹺���ҵ���˵��лл��

�������ԤԼ�ҵ� ����/���� ����ӭ[���ҷ����ʼ�](black@scene-si.org)�����ó� APIs��Go��Docker��VueJs ��ϵͳ���ݣ�[�Լ��ܶ�����������](https://scene-si.org/about)

----------------

via: https://scene-si.org/2018/02/07/sql-as-an-api/

���ߣ�[Tit Petric](https://scene-si.org/about)
���ߣ�[MoodWu](https://github.com/MoodWu)
У�ԣ�[У����ID](https://github.com/У����ID)

������ [GCTT](https://github.com/studygolang/GCTT) ԭ�����룬[Go ������](https://studygolang.com/) �����Ƴ�