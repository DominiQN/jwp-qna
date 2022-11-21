# QnA 서비스

# 요구사항

QnA 서비스를 만들어가면서 JPA로 실제 도메인 모델을 어떻게 구성하고 객체와 테이블을 어떻게 매핑해야 하는지 알아본다.

- 아래의 DDL(Data Definition Language)을 보고 유추하여 엔티티 클래스와 리포지토리 클래스를 작성해 본다.
    - [x] 사용자(User)
        - [x] DDL에 맞게 Entity 클래스 애노테이션 추가
        - [x] id 생성 테스트
        - [x] 길이 제약조건 테스트
        - [x] null 제약조건 테스트
        - [x] unique 제약조건 테스트
        - [x] 생성 및 수정일시 추가
        - [x] 손님인지 확인할 수 있다.
        - [x] 이름 및 이메일을 수정할 수 있다.
        - [x] 주어진 사용자와 이름과 이메일이 모두 일치하는지 확인할 수 있다.
    - [x] 질문(Question)
        - [x] DDL에 맞게 Entity 클래스 애노테이션 추가
        - [x] id 생성 테스트
        - [x] 길이 제약조건 테스트
        - [x] 생성 및 수정일시 추가
        - [x] 작성자를 수정할 수 있다.
        - [x] 주어진 사용자가 작성자인지 알 수 있다.
        - [x] 작성자를 사용자 엔티티와 매핑
        - [ ] 답변 목록을 조회할 수 있다. (답변 엔티티와 매핑)
    - [x] 답변(Answer)
        - [x] DDL에 맞게 Entity 클래스 애노테이션 추가
        - [x] id 생성 테스트
        - [x] 생성 및 수정일시 추가
        - [x] 작성자를 사용자 엔티티와 매핑
        - [ ] 질문을 알 수 있다. (질문 엔티티와 매핑)
    - [x] 삭제이력(DeleteHistory)
        - [x] DDL에 맞게 Entity 클래스 애노테이션 추가
        - [x] 저장 테스트
        - [x] 주어진 사용자가 작성자인지 확인할 수 있다.
        - [ ] 삭제한 사용자를 사용자 엔티티와 매핑
- @DataJpaTest를 사용하여 학습 테스트를 해 본다.

## DDL

```h2
create table answer
(
    id          bigint generated by default as identity,
    contents    clob,
    created_at  timestamp not null,
    deleted     boolean   not null,
    question_id bigint,
    updated_at  timestamp,
    writer_id   bigint,
    primary key (id)
);

create table delete_history
(
    id            bigint generated by default as identity,
    content_id    bigint,
    content_type  varchar(255),
    create_date   timestamp,
    deleted_by_id bigint,
    primary key (id)
);

create table question
(
    id         bigint generated by default as identity,
    contents   clob,
    created_at timestamp    not null,
    deleted    boolean      not null,
    title      varchar(100) not null,
    updated_at timestamp,
    writer_id  bigint,
    primary key (id)
);

create table user
(
    id         bigint generated by default as identity,
    created_at timestamp   not null,
    email      varchar(50),
    name       varchar(20) not null,
    password   varchar(20) not null,
    updated_at timestamp,
    user_id    varchar(20) not null,
    primary key (id)
);

alter table user
    add constraint UK_a3imlf41l37utmxiquukk8ajc unique (user_id);

alter table answer
    add constraint fk_answer_to_question
        foreign key (question_id)
            references question;

alter table answer
    add constraint fk_answer_writer
        foreign key (writer_id)
            references user;

alter table delete_history
    add constraint fk_delete_history_to_user
        foreign key (deleted_by_id)
            references user;

alter table question
    add constraint fk_question_writer
        foreign key (writer_id)
            references user;
```

## 힌트

```properties
spring.jpa.properties.hibernate.format_sql=true
spring.jpa.show-sql=true
```

[Auto-configured Data JPA Tests](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#features.testing.spring-boot-applications.autoconfigured-spring-data-jpa)

```java

@DataJpaTest
class StationRepositoryTest {
    @Autowired
    private StationRepository stations;

    @Test
    void save() {
        Station expected = new Station("잠실역");
        Station actual = stations.save(expected);
        assertAll(
                () -> assertThat(actual.getId()).isNotNull(),
                () -> assertThat(actual.getName()).isEqualTo(expected.getName())
        );
    }

    @Test
    void findByName() {
        String expected = "잠실역";
        stations.save(new Station(expected));
        String actual = stations.findByName(expected).getName();
        assertThat(actual).isEqualTo(expected);
    }
}
```

H2 데이터베이스를 사용한다면 아래의 프로퍼티를 추가하면 MySQL Dialect을 사용할 수 있다.

```properties
spring.datasource.url=jdbc:h2:~/test;MODE=MySQL
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL57Dialect
```

```mysql
create table answer
(
    id          bigint      not null auto_increment,
    contents    longtext,
    created_at  datetime(6) not null,
    deleted     bit         not null,
    question_id bigint,
    updated_at  datetime(6),
    writer_id   bigint,
    primary key (id)
) engine = InnoDB;

create table delete_history
(
    id            bigint not null auto_increment,
    content_id    bigint,
    content_type  varchar(255),
    create_date   datetime(6),
    deleted_by_id bigint,
    primary key (id)
) engine = InnoDB;

create table question
(
    id         bigint       not null auto_increment,
    contents   longtext,
    created_at datetime(6)  not null,
    deleted    bit          not null,
    title      varchar(100) not null,
    updated_at datetime(6),
    writer_id  bigint,
    primary key (id)
) engine = InnoDB;

create table user
(
    id         bigint      not null auto_increment,
    created_at datetime(6) not null,
    email      varchar(50),
    name       varchar(20) not null,
    password   varchar(20) not null,
    updated_at datetime(6),
    user_id    varchar(20) not null,
    primary key (id)
) engine = InnoDB;

alter table user
    add constraint UK_a3imlf41l37utmxiquukk8ajc unique (user_id);

alter table answer
    add constraint fk_answer_to_question
        foreign key (question_id)
            references question (id);

alter table answer
    add constraint fk_answer_writer
        foreign key (writer_id)
            references user (id);

alter table delete_history
    add constraint fk_delete_history_to_user
        foreign key (deleted_by_id)
            references user (id);

alter table question
    add constraint fk_question_writer
        foreign key (writer_id)
            references user (id);
```
