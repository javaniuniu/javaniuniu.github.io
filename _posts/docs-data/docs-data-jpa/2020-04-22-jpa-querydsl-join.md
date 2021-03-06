---
title: JPA通过querydsl做连表查询
permalink: /docs-data-jpa/querydsl/join
tags: JPA
key: docs-data-jpa-querydsl-join
---
## 结论：
- jpa只适合单表操作
- 通过 querydsl 实现复杂的sql （在复杂的连表面前也难办）
- jpa 是个大坑，做链表查询会把人累死
- 首选 mybitas
- 有哪个大神能做出来 记得@下
- 文章末尾有querydsl 相关配置

## 需要实现sql
```sql
select me.address as address, me.totalChecked as totalChecked, me.avatar as avatar, info.checked as checked, info.checkedTime as checkedTime, info.upvoteNumber as upvoteNumber, info.username as username
from (select member1.address, member1.totalChecked, member1.avatar, member1.username
from Member member1
where member1.status = 0) as me
left join (
select checkDayInfo.username, checkDayInfo.checked, checkDayInfo.checkedTime, checkDayInfo.upvoteNumber
from CheckDayInfo checkDayInfo
where checkDayInfo.date = '2020-04-17'
) as info
on me.username = info.username
order by info.checked desc, info.checkedTime asc
```

### 相关代码
```java
import com.querydsl.core.BooleanBuilder;
import com.querydsl.core.QueryModifiers;
import com.querydsl.core.QueryResults;
import com.querydsl.core.types.*;
import com.querydsl.core.types.dsl.*;
import com.querydsl.jpa.impl.JPAQuery;
import com.querydsl.jpa.impl.JPAQueryFactory;
import com.querydsl.jpa.sql.JPASQLQuery;
import com.querydsl.sql.DerbyTemplates;
import com.querydsl.sql.SQLExpressions;
import com.querydsl.sql.SQLTemplates;
import org.junit.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.domain.PageRequest;
import org.springframework.data.domain.Pageable;

import javax.annotation.PostConstruct;
import javax.persistence.EntityManager;
import java.util.List;
import java.util.stream.Collectors;


public class PageTest extends SpringbootJpaApplicationTests {

    @Autowired
    private EntityManager entityManager;

    //查询工厂实体
    private JPAQueryFactory queryFactory;


    //实例化控制器完成后执行该方法实例化JPAQueryFactory
    @PostConstruct
    public void initFactory() {
        queryFactory = new JPAQueryFactory(entityManager);
    }


    /**
     * 普通查询
     */
    @Test
    public void getcheckDayInfo() {
        QCheckDayInfo qCheckDayInfo = QCheckDayInfo.checkDayInfo;
        List<CheckDayInfo> checkDayInfo = queryFactory
                .selectFrom(qCheckDayInfo)
                .where(
                        qCheckDayInfo.username.eq("javaniuniu"),
                        qCheckDayInfo.checked.eq(1)
                )
                .fetch();
        System.out.println(checkDayInfo);
    }

    /**
     * BooleanBuilder 绑定条件进行查询
     */
    @Test
    public void getCheckDayInfos() {
        // 使用 QueryDSL 进行查询
        QCheckDayInfo qCheckDayInfo = QCheckDayInfo.checkDayInfo;
        // 定于获取条件
        BooleanBuilder booleanBuilder = new BooleanBuilder();
        // 放入要查询的条件信息
        booleanBuilder.and(qCheckDayInfo.username.contains("javaniuniu"));
        // queryFactory 是上方定义的工厂实体
        // select（生成的实体类的字段）.from（生成实体类的名称）.where（上方要查询的条件）.orderBy（排序）.fetch（）进行查询
        List<CheckDayInfo> checkDayInfoList = queryFactory.select(qCheckDayInfo)
                .from(qCheckDayInfo)
                .where(booleanBuilder)
                .orderBy(qCheckDayInfo.date.asc())
                .fetch();
        System.out.println(checkDayInfoList);
    }

    /**
     * 分页查询
     */
    @Test
    public void getCheckDayInfoPage() {
        Pageable pageable = PageRequest.of(0, 5);
        QCheckDayInfo qCheckDayInfo = QCheckDayInfo.checkDayInfo;
        QueryResults<CheckDayInfo> checkDayInfo = queryFactory
                .selectFrom(qCheckDayInfo)
                .orderBy(qCheckDayInfo.checked.desc())
                .orderBy(qCheckDayInfo.checkedTime.desc())
                .offset(pageable.getOffset())
                .limit(pageable.getPageSize())
                .fetchResults();
        System.out.println(checkDayInfo);
    }

    /**
     * 通过 JPASQLQuery 做分页查询
     */
    @Test
    public void getbCheckDayInfoPageByJPASQLQuery() {
        SQLTemplates templates = new DerbyTemplates();

        JPASQLQuery<?> jpasqlQuery = new JPASQLQuery<Void>(entityManager, templates);
        Pageable pageable = PageRequest.of(0, 5);
        QCheckDayInfo qCheckDayInfo = QCheckDayInfo.checkDayInfo;
        QueryResults<CheckDayInfo> checkDayInfo = jpasqlQuery
                .select(qCheckDayInfo)
                .from(qCheckDayInfo)
                .orderBy(qCheckDayInfo.checked.desc())
                .orderBy(qCheckDayInfo.checkedTime.desc())
                .offset(pageable.getOffset())
                .limit(pageable.getPageSize())
                .fetchResults();
        System.out.println(checkDayInfo);
    }

    /**
     * 返回数据到DTO
     */
    @Test
    public void getCheckDayInfoDTO() {
        QCheckDayInfo qCheckDayInfo = QCheckDayInfo.checkDayInfo;
        Pageable pageable = PageRequest.of(0, 2);
        List<CheckDayInfoDTO> checkDayInfo = queryFactory
                .select(
                        qCheckDayInfo.username,
                        qCheckDayInfo.checked,
                        qCheckDayInfo.checkedTime,
                        qCheckDayInfo.upvoteNumber
                )
                .from(qCheckDayInfo)
//                .where(
//                        qCheckDayInfo.checked.eq(1)
//                )
                .orderBy(qCheckDayInfo.checked.desc())
                .orderBy(qCheckDayInfo.checkedTime.asc())
                .offset(pageable.getOffset())
                .limit(pageable.getPageSize())
                .fetch()
                .stream()
                .map(tuple -> CheckDayInfoDTO.builder()
                        .username(tuple.get(qCheckDayInfo.username))
                        .checked(tuple.get(qCheckDayInfo.checked))
                        .checkedTime(tuple.get(qCheckDayInfo.checkedTime))
                        .upvoteNumber(tuple.get(qCheckDayInfo.upvoteNumber))
                        .build()
                )
                .collect(Collectors.toList());
        System.out.println(checkDayInfo);
    }


    /**
     * 通过 JPAQuery 做连表查询
     * 但是不能分页
     */
    @Test
    public void getCheckDayInfoDTOPages() {

        SQLTemplates templates = new DerbyTemplates();

        JPASQLQuery<?> jpasqlQuery = new JPASQLQuery<Void>(entityManager, templates);

        JPAQuery jpaQuery = new JPAQuery(entityManager);
        Pageable pageable = PageRequest.of(1, 2);

        String date = "2020-04-17";
        QMember qMember = QMember.member;
        QCheckDayInfo qCheckDayInfo = QCheckDayInfo.checkDayInfo;

        StringPath me = Expressions.stringPath("me");
        StringPath info = Expressions.stringPath("info");


        SimpleTemplate<String> address = Expressions.template(String.class, "me.address");
        NumberTemplate<Integer> totalChecked = Expressions.numberTemplate(Integer.class, "me.totalChecked");
        SimpleTemplate<String> avatar = Expressions.template(String.class, "me.avatar");
        SimpleTemplate<String> me_username = Expressions.template(String.class, "me.username");

        NumberTemplate<Integer> checked = Expressions.numberTemplate(Integer.class, "info.checked");
        SimpleTemplate<String> checkedTime = Expressions.template(String.class, "info.checkedTime");
        NumberTemplate<Integer> upvoteNumber = Expressions.numberTemplate(Integer.class, "info.upvoteNumber");
        SimpleTemplate<String> username = Expressions.template(String.class, "info.username");

        OrderSpecifier orderByChecked = new OrderSpecifier(Order.DESC,
                Expressions.template(String.class, "info.checked"));

        OrderSpecifier orderByCheckedTime = new OrderSpecifier(Order.ASC,
                Expressions.template(String.class, "info.checkedTime"));

//        long l1 = 2;
//        long l2 = 1;
//
//        QueryModifiers queryModifiers = new QueryModifiers(l1, l2);

        SubQueryExpression queryCheckByDate = SQLExpressions
                .select(
                        qCheckDayInfo.username,
                        qCheckDayInfo.checked,
                        qCheckDayInfo.checkedTime,
                        qCheckDayInfo.upvoteNumber
                )
                .from(qCheckDayInfo)
                .where(qCheckDayInfo.date.eq(date));

        SubQueryExpression queryMember = SQLExpressions
                .select(
                        qMember.address,
                        qMember.totalChecked,
                        qMember.avatar,
                        qMember.username
                )
                .from(qMember)
                .where(qMember.status.eq(0));

        QueryResults<CheckDayInfoDTO> checkDayInfoDTOS = jpasqlQuery.select(
                Projections.bean(
                        CheckDayInfoDTO.class,
                        address.as("address"), totalChecked.as("totalChecked"), avatar.as("avatar"), checked.as("checked"),
                        checkedTime.as("checkedTime"),
                        upvoteNumber.as("upvoteNumber"), username.as("username")))
                .from(qMember)
                .leftJoin(queryCheckByDate, info).on(me_username.eq(username))
                .orderBy(orderByChecked)
                .orderBy(orderByCheckedTime)
//                .limit(pageable.getPageSize()) // 做不了分页
//                .offset(pageable.getOffset())
//                .restrict(queryModifiers)
                .fetchResults();
        System.out.println(checkDayInfoDTOS);


    }

}

```

### querydsl 相关配置
```xml
<!-- QueryDSL 相关依赖 -->
        <dependency>
            <groupId>com.querydsl</groupId>
            <artifactId>querydsl-jpa</artifactId>
            <version>4.1.4</version>
        </dependency>
        <dependency>
            <groupId>com.querydsl</groupId>
            <artifactId>querydsl-apt</artifactId>
            <version>4.3.1</version>
            <scope>provided</scope>
        </dependency>

        <!-- https://mvnrepository.com/artifact/com.querydsl/querydsl-sql -->
        <dependency>
            <groupId>com.querydsl</groupId>
            <artifactId>querydsl-sql</artifactId>
            <version>4.3.1</version>
        </dependency>
<!--         https://mvnrepository.com/artifact/com.mysema.querydsl/querydsl-jdo-->
        <dependency>
            <groupId>com.mysema.querydsl</groupId>
            <artifactId>querydsl-jdo</artifactId>
            <version>3.7.4</version>
        </dependency>

        <!-- https://mvnrepository.com/artifact/com.mysema.querydsl/querydsl-apt -->
        <dependency>
            <groupId>com.mysema.querydsl</groupId>
            <artifactId>querydsl-sql</artifactId>
            <version>3.7.4</version>
            <scope>provided</scope>
        </dependency>


        ...

        <!--因为是类型安全的，所以还需要加上Maven APT plugin，使用 APT 自动生成一些类:-->
            <plugin>
                <groupId>com.mysema.maven</groupId>
                <artifactId>apt-maven-plugin</artifactId>
                <version>1.1.3</version>
                <executions>
                    <execution>
                        <phase>generate-sources</phase>
                        <goals>
                            <goal>process</goal>
                        </goals>
                        <configuration>
                            <outputDirectory>target/generated-sources</outputDirectory>
                            <processor>com.querydsl.apt.jpa.JPAAnnotationProcessor</processor>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
```

### 生成 Q实体类

![pic](/assets/images/docs-data/docs-data-jpa/0425/2019071014215688.png)

### 参考文档
- [关于 QueryDSL 配置和使用（详细](https://blog.csdn.net/qq_36537546/article/details/95315040)
