# mybatis-generator-core-oracle

## mybatis gererator 真的Oracle定制，实现分页，ID自增，返回ID等功能。

主要针对Oracle数据库定制，添加功能列表如下：
- 分页
- 自增
- 返回自增ID

### 添加分页功能
**1** 修改`Mapping`文件（文件位置：src\main\java\org\mybatis\generator\codegen\mybatis3\xmlmapper\elements\）

- SelectByExampleWithBLOBsElementGenerator.java
- SelectByExampleWithoutBLOBsElementGenerator.java

添加内容以`SelectByExampleWithBLOBsElementGenerator.java`文件为例，`SelectByExampleWithoutBLOBsElementGenerator.java`文件的修改相同：

```java
#48行，请参考具体文件内容
XmlElement ifElement = new XmlElement("if");
ifElement.addAttribute(new Attribute("test", "start != 0 or limit != 0"));
ifElement.addElement(new TextElement("select * from ( select * from ("));
```

修改`' as QUERYID,` 为 `' as QUERYID,ROWNUM as RN,`，使用ROWNUM进行分页
```java
StringBuilder sb = new StringBuilder();
        if (stringHasValue(introspectedTable
                .getSelectByExampleQueryId())) {
            sb.append('\'');
            sb.append(introspectedTable.getSelectByExampleQueryId());
            sb.append("' as QUERYID,ROWNUM as RN,"); //$NON-NLS-1$
            answer.addElement(new TextElement(sb.toString()));
        }
```

```java
#85行，请参考具体文件内容
ifElement = new XmlElement("if");
ifElement.addAttribute(new Attribute("test", "start != 0 or limit != 0"));
ifElement.addElement(new TextElement(") A where A.RN &lt;= #{limit} ) B where B.RN &gt; #{start}"));
answer.addElement(ifElement);
```

生成Mapping.xml文件中select语句为：

```xml
<select id="selectByExampleWithBLOBs" resultMap="ResultMapWithBLOBs" parameterType="com.cmcc.emp.model.AlgoExample" >
    <if test="start != 0 or limit != 0" >
      select * from ( select * from (
    </if>
    select
    <if test="distinct" >
      distinct
    </if>
    'true' as QUERYID,ROWNUM as RN,
    <include refid="Base_Column_List" />
    ,
    <include refid="Blob_Column_List" />
    from EMP_ALGO
    <if test="_parameter != null" >
      <include refid="Example_Where_Clause" />
    </if>
    <if test="orderByClause != null" >
      order by ${orderByClause}
    </if>
    <if test="start != 0 or limit != 0" >
      ) A where A.RN &lt;= #{limit} ) B where B.RN &gt; #{start}
    </if>
  </select>
```

2 修改`model`文件（文件位置：src\main\java\org\mybatis\generator\codegen\mybatis3\model\）

- ExampleGenerator.java

添加构造方法`public XxxExample(int start, int limit)`

```java
#73行，请参考具体文件内容
method = new Method();
        method.setVisibility(JavaVisibility.PUBLIC);
        method.setConstructor(true);
        method.setName(type.getShortName());
        method.addParameter(new Parameter(FullyQualifiedJavaType
                .getIntInstance(), "start"));
        method.addParameter(new Parameter(FullyQualifiedJavaType
                .getIntInstance(), "limit"));
        method.addBodyLine("this.start = start;\n" +
                "        this.limit = limit;\n" +
                "        oredCriteria = new ArrayList<Criteria>();");

        commentGenerator.addGeneralMethodComment(method, introspectedTable);
        topLevelClass.addMethod(method);
```

添加参数：`protected int start;` 和 `protected int limit;`

```java
#140行，请参考具体文件内容
field = new Field();
        field.setVisibility(JavaVisibility.PROTECTED);
        field.setType(FullyQualifiedJavaType.getIntInstance());
        field.setName("start");
        commentGenerator.addFieldComment(field, introspectedTable);
        topLevelClass.addField(field);

        field = new Field();
        field.setVisibility(JavaVisibility.PROTECTED);
        field.setType(FullyQualifiedJavaType.getIntInstance());
        field.setName("limit");
        commentGenerator.addFieldComment(field, introspectedTable);
        topLevelClass.addField(field);
```

生成Model文件中内容为，和上面的select语句中的start、limit关联上了：

```java
...
protected int start;

    protected int limit;

    protected List<Criteria> oredCriteria;

    public AlgoExample() {
        oredCriteria = new ArrayList<Criteria>();
    }

    public AlgoExample(int start, int limit) {
        this.start = start;
        this.limit = limit;
        oredCriteria = new ArrayList<Criteria>();
    }
...
```




