# XML 全面解析与实战指南

## 从零开始掌握XML

### 1. XML核心概念
#### 1.1 数据标记的进化史
XML（eXtensible Markup Language）是一种类似HTML的标记语言，但专注于数据存储和传输。想象XML就像乐高积木：
- 每个标签都是独特的积木块（如`<address>`）
- 积木可以自由组合（嵌套结构）
- 积木颜色形状自定（可扩展性）

示例：简单的联系人XML
```xml
<联系人>
  <姓名>张三</姓名>
  <电话 type="手机">13800138000</电话>
  <地址>
    <城市>北京</城市>
    <街道>中关村大街1号</街道>
  </地址>
</联系人>
```

#### 1.2 XML与HTML的本质差异
|| XML | HTML |
|---|---|---|
|目的|数据承载|内容展示|
|标签|自定义|预定义|
|大小写|敏感|不敏感|
|错误处理|严格|宽松|

### 2. XML基础语法
#### 2.1 文档结构规范
##### 2.1.1 XML声明头
```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
```
- version：必须首位出现
- encoding：推荐使用UTF-8
- standalone：是否依赖外部DTD

##### 2.1.2 元素树结构
```xml
<图书目录>
  <图书 ISBN="9787115547">
    <书名>XML实战指南</书名>
    <价格 货币="CNY">89.00</价格>
  </图书>
</图书目录>
```
*根元素`<图书目录>`包含整个文档内容*

#### 2.2 命名规范详解
1. 合法命名示例：`<Book123>`, `<中文标签>`
2. 非法命名示例：`<123Book>`, `<xml-Tag>`
3. 最佳实践：使用英文小写+下划线（如`<book_title>`）

#### 2.3 特殊内容处理
##### 2.3.1 CDATA区块
```xml
<代码示例>
<![CDATA[
function compare(a, b) {
    return a < b ? -1 : (a > b ? 1 : 0);
}
]]>
</代码示例>
```
*避免`<`和`>`被解析为标签*

##### 2.3.2 实体引用
| 字符 | 实体引用 |
|---|---|
| < | `&lt;` |
| > | `&gt;` |
| & | `&amp;` |
| " | `&quot;` |
| ' | `&apos;` |

### 3. XML约束机制
#### 3.1 DTD文档类型定义
```xml
<!DOCTYPE 班级 [
  <!ELEMENT 班级 (学生+)>
  <!ELEMENT 学生 (姓名,年龄)>
  <!ELEMENT 姓名 (#PCDATA)>
  <!ELEMENT 年龄 (#PCDATA)>
  <!ATTLIST 学生 学号 ID #REQUIRED>
]>
```
*定义元素关系和属性约束*

#### 3.2 XML Schema进阶
```xml
<xs:schema xmlns:xs="http://www.w3.org/2001/XMLSchema">
  <xs:element name="图书">
    <xs:complexType>
      <xs:sequence>
        <xs:element name="书名" type="xs:string"/>
        <xs:element name="价格" type="xs:decimal"/>
      </xs:sequence>
      <xs:attribute name="ISBN" type="xs:string" use="required"/>
    </xs:complexType>
  </xs:element>
</xs:schema>
```
*支持数据类型校验和复杂约束*

### 4. XML处理技术
#### 4.1 DOM解析模型
```java
DocumentBuilderFactory factory = DocumentBuilderFactory.newInstance();
DocumentBuilder builder = factory.newDocumentBuilder();
Document doc = builder.parse("data.xml");

NodeList books = doc.getElementsByTagName("图书");
for (int i=0; i<books.getLength(); i++) {
    Element book = (Element)books.item(i);
    String title = book.getElementsByTagName("书名").item(0).getTextContent();
}
```
*适合小文件随机访问*

#### 4.2 SAX事件驱动
```java
SAXParserFactory factory = SAXParserFactory.newInstance();
SAXParser parser = factory.newSAXParser();

DefaultHandler handler = new DefaultHandler() {
    boolean inTitle = false;
    
    public void startElement(String uri, String localName, String qName, Attributes attributes) {
        if (qName.equalsIgnoreCase("书名")) {
            inTitle = true;
        }
    }
    
    public void characters(char[] ch, int start, int length) {
        if (inTitle) {
            System.out.println("书名：" + new String(ch, start, length));
        }
    }
};
```
*适合大文件流式处理*

### 5. 现代XML应用
#### 5.1 配置文件实践
```xml
<!-- Spring配置示例 -->
<beans xmlns="http://www.springframework.org/schema/beans">
  <bean id="dataSource" class="com.mysql.jdbc.Driver">
    <property name="url" value="jdbc:mysql://localhost:3306/mydb"/>
    <property name="username" value="root"/>
    <property name="password" value="123456"/>
  </bean>
</beans>
```

#### 5.2 数据交换标准
- SOAP WebService
- RSS订阅源
- SVG矢量图形
- Office Open XML

### 6. 安全注意事项
1. XXE（XML外部实体）攻击防护
2. DTD拒绝服务攻击预防
3. 实体扩展炸弹防范
4. 敏感信息加密处理
