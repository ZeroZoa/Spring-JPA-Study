# Spring-JPA-Study
ZeroZoa의 스프링 JPA 공부 기록

<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>JPA Study Notes</title>
    <style>
        body {
            font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Helvetica, Arial, sans-serif, "Apple Color Emoji", "Segoe UI Emoji";
            line-height: 1.6;
            color: #24292e;
            max-width: 800px;
            margin: 0 auto;
            padding: 20px;
        }
        h1, h2, h3 {
            border-bottom: 1px solid #eaecef;
            padding-bottom: .3em;
            margin-top: 24px;
            margin-bottom: 16px;
            font-weight: 600;
        }
        h1 { font-size: 2em; }
        h2 { font-size: 1.5em; }
        h3 { font-size: 1.25em; }
        p, ul, ol {
            margin-bottom: 16px;
        }
        li {
            margin-bottom: 8px;
        }
        code {
            font-family: "SFMono-Regular", Consolas, "Liberation Mono", Menlo, Courier, monospace;
            background-color: rgba(27,31,35,.05);
            padding: .2em .4em;
            margin: 0;
            font-size: 85%;
            border-radius: 3px;
        }
        pre code {
            display: block;
            padding: 16px;
            overflow: auto;
            font-size: 85%;
            line-height: 1.45;
            border-radius: 3px;
        }
        blockquote {
            margin-left: 0;
            padding-left: 1em;
            border-left: .25em solid #dfe2e5;
            color: #6a737d;
        }
        hr {
            height: .25em;
            padding: 0;
            margin: 24px 0;
            background-color: #e1e4e8;
            border: 0;
        }
        a {
            color: #0366d6;
            text-decoration: none;
        }
        a:hover {
            text-decoration: underline;
        }
        strong {
            font-weight: 600;
        }
    </style>
</head>
<body>

    <h1>📚 JPA Study Notes</h1>
    <p>이 저장소는 Java Persistence API (JPA)를 학습하고 정리한 내용을 담고 있습니다. ORM의 기본 개념부터 엔티티 매핑, 영속성 관리, 연관관계, 프록시, 값 타입 등 JPA의 핵심적인 내용을 다룹니다.</p>
    <br>

    <h2>📌 JPA 실무 Best Practice</h2>
    <p>실무에서 JPA를 효과적으로 사용하기 위한 핵심 전략입니다.</p>
    <ol>
        <li><strong>🚀 모든 연관관계는 지연 로딩(<code>LAZY</code>)으로 설정하세요.</strong>
            <ul>
                <li>즉시 로딩(<code>EAGER</code>)은 예측하지 못한 SQL 문제를 일으키는 주범이며, 특히 JPQL에서 N+1 문제를 발생시킬 수 있습니다.</li>
                <li><code>@ManyToOne</code>, <code>@OneToOne</code> 관계는 기본값이 즉시 로딩이므로 반드시 <code>fetch = FetchType.LAZY</code> 옵션을 명시해야 합니다.</li>
                <li>성능 최적화가 필요한 경우 <strong>Fetch Join</strong>이나 <strong><code>@EntityGraph</code></strong>, <strong><code>@BatchSize</code></strong>를 사용해 필요한 데이터만 함께 조회하는 것을 권장합니다.</li>
            </ul>
        </li>
        <li><strong>🔗 외래 키(FK)가 있는 곳을 연관관계의 주인으로 정하세요.</strong>
            <ul>
                <li>연관관계의 주인은 외래 키를 직접 관리하는 엔티티가 되어야 합니다.</li>
                <li>주인이 아닌 곳에 외래 키를 매핑하면, 연관관계 관리를 위해 추가적인 UPDATE 쿼리가 발생하여 성능 저하의 원인이 됩니다.</li>
            </ul>
        </li>
        <li><strong>🛡️ 엔티티에는 가급적 <code>@Setter</code>를 사용하지 마세요.</strong>
            <ul>
                <li><code>@Setter</code>를 무분별하게 사용하면 엔티티의 변경 지점을 추적하기 어려워 유지보수가 힘들어집니다.</li>
                <li>대신, <strong>변경 목적이 명확한 비즈니스 메서드</strong>를 별도로 만들어 관리하는 것이 좋습니다. (예: <code>cancelOrder()</code>, <code>updateStock(int stock)</code>)</li>
            </ul>
        </li>
        <li><strong>💎 값 타입(Embeddable)은 불변 객체(Immutable Object)로 설계하세요.</strong>
            <ul>
                <li>객체 타입은 참조를 공유하기 때문에, 여러 엔티티에서 하나의 값 타입을 공유하면 예상치 못한 부작용(Side Effect)이 발생할 수 있습니다.</li>
                <li><code>@Setter</code>를 제거하고, 생성 시점에만 값을 할당하여 변경 불가능하게 만들면 안전하게 사용할 수 있습니다.</li>
            </ul>
        </li>
        <li><strong>✨ 컬렉션은 필드에서 즉시 초기화하세요.</strong>
            <ul>
                <li><code>NullPointerException</code>과 같은 예기치 않은 오류를 방지하기 위해, 컬렉션은 선언과 동시에 초기화하는 것이 안전합니다.</li>
                <li><code>List&lt;Comment&gt; comments = new ArrayList&lt;&gt;();</code></li>
            </ul>
        </li>
        <li><strong>📤 API를 만들 때 엔티티를 외부로 직접 반환하지 마세요.</strong>
            <ul>
                <li>엔티티는 데이터베이스와 직접 연결된 핵심 도메인 객체입니다. 엔티티가 변경되면 API 스펙 전체가 흔들릴 수 있습니다.</li>
                <li>반드시 <strong>DTO(Data Transfer Object)를 사용</strong>하여 API 응답에 필요한 데이터만 선별적으로 노출하세요. 민감한 정보 유출도 막을 수 있습니다.</li>
            </ul>
        </li>
    </ol>
    <br>
    <hr>
    
    <h2>📖 Table of Contents</h2>
    <ol>
        <li><a href="#ch1">JPA 소개</a></li>
        <li><a href="#ch2">JPA 시작하기</a></li>
        <li><a href="#ch3">영속성 관리</a></li>
        <li><a href="#ch4">엔티티 매핑</a></li>
        <li><a href="#ch5">연관관계 매핑 기초</a></li>
        <li><a href="#ch6">다양한 연관관계 매핑</a></li>
        <li><a href="#ch7">프록시와 연관관계 관리</a></li>
        <li><a href="#ch8">값 타입</a></li>
        <li><a href="#ch9">객체지향 쿼리 언어 (JPQL)</a></li>
    </ol>
    <br>

    <h2 id="ch1">📜 Chapter 1 : JPA 소개</h2>
    <h3>ORM (Object Relational Mapping)</h3>
    <ul>
        <li>이름 그대로 <strong>객체</strong>와 <strong>관계형 데이터베이스</strong>를 매핑하는 기술입니다.</li>
        <li>ORM 프레임워크가 객체와 테이블 간의 패러다임 불일치 문제를 해결해 줍니다.</li>
    </ul>
    <h3>JPA (Java Persistence API)</h3>
    <ul>
        <li>Java 진영의 ORM 기술 표준 명세(Interface)입니다.</li>
        <li>JPA의 구현체로는 Hibernate, EclipseLink 등이 있으며, 우리는 주로 Hibernate를 사용하게 됩니다.</li>
    </ul>
    <h3>JPA의 특징</h3>
    <ul>
        <li><strong>생산성</strong>: <code>save()</code>, <code>findById()</code> 등 기본적인 CRUD 메서드를 제공하여 반복적인 SQL 작업을 줄여줍니다. DDL 자동 생성 기능도 지원합니다.</li>
        <li><strong>유지보수</strong>: 필드 변경 시 모든 SQL을 수정할 필요 없이, 매핑 정보만 수정하면 되어 유지보수가 용이합니다.</li>
        <li><strong>패러다임 불일치 해결</strong>: 상속, 연관관계, 객체 그래프 탐색 등 객체와 관계형 DB의 차이에서 오는 문제를 해결합니다.</li>
        <li><strong>벤더 독립성</strong>: 추상화된 데이터 접근 계층을 제공하므로, 데이터베이스를 변경해도 비즈니스 로직 코드의 변경을 최소화할 수 있습니다.</li>
        <li><strong>성능 최적화</strong>: 애플리케이션과 DB 사이에서 1차 캐시, 쓰기 지연, 지연 로딩 등을 통해 성능 최적화를 시도할 수 있습니다.</li>
    </ul>
    <br>
    
    <h2 id="ch2">🚀 Chapter 2 : JPA 시작하기</h2>
    <h3>1. 객체 매핑 어노테이션</h3>
    <ul>
        <li><code>@Entity</code>: 클래스를 테이블과 매핑한다고 JPA에게 알려줍니다.</li>
        <li><code>@Table</code>: 엔티티와 매핑할 테이블 정보를 지정합니다. (기본값: 클래스 이름)</li>
        <li><code>@Id</code>: 필드를 테이블의 기본 키(Primary Key)에 매핑합니다.</li>
        <li><code>@Column</code>: 필드를 테이블의 컬럼에 매핑합니다.</li>
    </ul>
    <h3>2. persistence.xml (Spring Boot에서는 <code>application.yml</code>)</h3>
    <ul>
        <li>JPA가 동작하는 데 필요한 설정 파일입니다.</li>
        <li>데이터베이스 드라이버, 접속 URL, 계정 정보, 데이터베이스 방언(Dialect) 등을 설정합니다.</li>
    </ul>
    <h3>3. 애플리케이션 개발</h3>
    <ul>
        <li><strong><code>EntityManagerFactory</code></strong>:
            <ul>
                <li>설정 정보를 바탕으로 생성되며, 생성 비용이 매우 크므로 애플리케이션 전체에서 <strong>단 하나만 생성</strong>하여 공유합니다. (Thread-safe)</li>
            </ul>
        </li>
        <li><strong><code>EntityManager</code></strong>:
            <ul>
                <li>팩토리에서 생성되며, 실제 JPA의 기능을 대부분 수행합니다.</li>
                <li><strong>동시성 문제</strong>가 발생할 수 있으므로 여러 스레드 간에 절대 공유하면 안 됩니다.</li>
            </ul>
        </li>
        <li><strong>트랜잭션 관리</strong>:
            <ul>
                <li>JPA를 통해 데이터를 변경하는 모든 작업은 반드시 <strong>트랜잭션 안에서</strong> 수행되어야 합니다.</li>
            </ul>
        </li>
    </ul>
    <br>

    <h2 id="ch3">✨ Chapter 3 : 영속성 관리</h2>
    <h3>영속성 컨텍스트 (Persistence Context)</h3>
    <ul>
        <li>"엔티티를 영구 저장하는 환경"으로, 엔티티 매니저를 통해 접근하고 관리합니다.</li>
        <li>애플리케이션과 데이터베이스 사이에서 객체를 보관하는 <strong>가상의 데이터베이스</strong> 역할을 합니다.</li>
    </ul>
    <h3>엔티티의 생명주기 (Entity Lifecycle)</h3>
    <ol>
        <li><strong>비영속 (New/Transient)</strong>: 순수한 객체 상태. 영속성 컨텍스트와 관계가 없습니다.</li>
        <li><strong>영속 (Managed)</strong>: <code>entityManager.persist()</code>를 통해 영속성 컨텍스트에 저장된 상태.</li>
        <li><strong>준영속 (Detached)</strong>: 영속성 컨텍스트가 관리하던 엔티티를 더 이상 관리하지 않는 상태. (<code>detach()</code>, <code>clear()</code>, <code>close()</code>)</li>
        <li><strong>삭제 (Removed)</strong>: 엔티티를 영속성 컨텍스트와 데이터베이스에서 삭제한 상태.</li>
    </ol>
    <h3>영속성 컨텍스트의 장점</h3>
    <ul>
        <li><strong>1차 캐시</strong>: 영속성 컨텍스트 내부에 캐시를 두어, 같은 트랜잭션 안에서 같은 엔티티를 조회하면 DB를 통하지 않고 캐시에서 반환합니다.</li>
        <li><strong>동일성 보장</strong>: 1차 캐시 덕분에 같은 트랜잭션 내에서 조회한 같은 엔티티의 참조값은 항상 동일합니다. <code>a == b</code></li>
        <li><strong>트랜잭션을 지원하는 쓰기 지연 (Transactional Write-behind)</strong>:
            <ul>
                <li><code>persist()</code> 시점에 바로 INSERT SQL을 보내지 않고, 내부 SQL 저장소에 쿼리를 모아둡니다.</li>
                <li>트랜잭션을 <code>commit()</code> 하는 시점에 모아둔 쿼리를 한 번에 데이터베이스로 보냅니다.</li>
            </ul>
        </li>
        <li><strong>변경 감지 (Dirty Checking)</strong>:
            <ul>
                <li>트랜잭션 커밋 시, 1차 캐시에 저장된 최초 상태(스냅샷)와 현재 엔티티를 비교하여 변경된 내용을 자동으로 UPDATE 쿼리로 만들어 실행합니다.</li>
                <li>덕분에 개발자는 <code>update()</code> 메서드 없이, 객체의 상태만 변경하면 됩니다.</li>
            </ul>
        </li>
        <li><strong>지연 로딩 (Lazy Loading)</strong>: Chapter 7에서 자세히 설명.</li>
    </ul>
    <h3>플러시 (Flush)</h3>
    <ul>
        <li>영속성 컨텍스트의 변경 내용을 데이터베이스에 동기화하는 작업입니다.</li>
        <li>쓰기 지연 SQL 저장소의 쿼리가 DB로 전송됩니다.</li>
        <li>플러시가 발생해도 1차 캐시는 유지됩니다.</li>
        <li><strong>플러시 호출 시점</strong>:
            <ol>
                <li><code>entityManager.flush()</code> 직접 호출</li>
                <li>트랜잭션 커밋 시 자동 호출</li>
                <li>JPQL 쿼리 실행 시 자동 호출</li>
            </ol>
        </li>
    </ul>
    <br>

    <h2 id="ch4">📝 Chapter 4 : 엔티티 매핑</h2>
    <h3>기본 키(PK) 매핑 전략 (<code>@GeneratedValue</code>)</h3>
    <ul>
        <li><strong><code>IDENTITY</code></strong>:
            <ul>
                <li>기본 키 생성을 데이터베이스에 위임합니다. (예: MySQL의 <code>AUTO_INCREMENT</code>)</li>
                <li><code>persist()</code> 시점에 즉시 INSERT 쿼리가 실행되고 DB에서 식별자를 받아옵니다.</li>
            </ul>
        </li>
        <li><strong><code>SEQUENCE</code></strong>:
            <ul>
                <li>데이터베이스 시퀀스 오브젝트를 사용하여 기본 키를 할당합니다. (예: Oracle, PostgreSQL)</li>
                <li><code>@SequenceGenerator</code>를 통해 시퀀스 정보를 상세히 설정할 수 있습니다.</li>
                <li><code>allocationSize</code> 옵션을 통해 미리 여러 식별자를 받아와 성능을 최적화할 수 있습니다.</li>
            </ul>
        </li>
        <li><strong><code>TABLE</code></strong>:
            <ul>
                <li>키 생성 전용 테이블을 만들어 시퀀스를 흉내 내는 전략입니다.</li>
                <li>모든 데이터베이스에서 사용 가능하지만, 성능상 단점이 있어 잘 사용되지 않습니다.</li>
            </ul>
        </li>
        <li><strong><code>AUTO</code> (기본값)</strong>:
            <ul>
                <li>연결된 데이터베이스 방언에 따라 위 세 가지 전략 중 하나를 자동으로 선택합니다.</li>
            </ul>
        </li>
    </ul>
    <blockquote>💡 <strong>권장 식별자 선택 전략</strong><br>
    비즈니스와 무관한 <strong>대리 키(Surrogate Key)</strong> 사용을 권장합니다. 비즈니스 요구사항은 변할 수 있기 때문에, 주민등록번호나 이메일 같은 자연 키(Natural Key)를 기본 키로 사용하는 것은 위험합니다. <code>Long</code> 타입의 자동 생성 전략이 가장 무난합니다.
    </blockquote>
    <br>

    <h2 id="ch5">🔗 Chapter 5 & 6 : 다양한 연관관계 매핑</h2>
    <h3>연관관계 매핑의 핵심 키워드</h3>
    <ul>
        <li><strong>방향</strong>: 단방향, 양방향</li>
        <li><strong>다중성</strong>: 다대일(<code>@ManyToOne</code>), 일대다(<code>@OneToMany</code>), 일대일(<code>@OneToOne</code>), 다대다(<code>@ManyToMany</code>)</li>
        <li><strong>연관관계의 주인 (Owner)</strong>: 양방향 관계에서 외래 키를 관리하는 쪽.</li>
    </ul>
    <h3>연관관계의 주인</h3>
    <ul>
        <li>객체는 양방향 관계를 위해 서로 다른 단방향 참조 2개를 갖지만, 테이블은 외래 키 하나로 양방향 관계를 맺습니다. 이 차이를 해결하기 위해 <strong>연관관계의 주인</strong>을 정합니다.</li>
        <li><strong>주인만이 외래 키를 등록, 수정, 삭제할 수 있습니다.</strong> 주인이 아닌 쪽은 조회만 가능합니다.</li>
        <li>주인이 아닌 쪽은 <code>mappedBy</code> 속성을 사용하여 주인을 명시해야 합니다.</li>
        <li><strong>외래 키가 있는 곳을 주인으로 정하는 것이 규칙입니다.</strong></li>
    </ul>
    <h3>다대일 (N:1, <code>@ManyToOne</code>)</h3>
    <ul>
        <li><strong>가장 많이 사용되고 가장 권장되는 연관관계입니다.</strong></li>
        <li>외래 키가 항상 '다(N)' 쪽에 있으므로, <code>@ManyToOne</code>이 설정된 엔티티가 항상 연관관계의 주인이 됩니다.</li>
    </ul>
    <h3>일대다 (1:N, <code>@OneToMany</code>)</h3>
    <ul>
        <li>'일' 쪽에서 '다' 쪽을 참조하는 관계입니다.</li>
        <li><strong>일대다 단방향 관계는 추천되지 않습니다.</strong> 연관관계 관리를 위해 불필요한 UPDATE 쿼리가 발생하고, 유지보수가 어렵기 때문입니다.</li>
        <li>대신, <strong>다대일 양방향 관계</strong>를 사용하는 것이 좋습니다.</li>
    </ul>
    <h3>일대일 (1:1, <code>@OneToOne</code>)</h3>
    <ul>
        <li>주 테이블이나 대상 테이블 중 원하는 곳에 외래 키를 둘 수 있습니다.</li>
        <li><strong>주로 접근하는 테이블에 외래 키를 두는 것</strong>이 일반적이며, 이 테이블이 연관관계의 주인이 됩니다.</li>
    </ul>
    <h3>다대다 (N:M, <code>@ManyToMany</code>)</h3>
    <ul>
        <li><code>@ManyToMany</code>는 편리해 보이지만, 연결 테이블에 추가적인 컬럼을 가질 수 없고, 예상치 못한 쿼리를 발생시키는 등 문제가 많아 <strong>실무에서는 사용하지 않습니다.</strong></li>
        <li>대신, 연결 테이블을 위한 <strong>엔티티를 별도로 생성</strong>하고, <strong>다대일-일대다 관계</strong>로 풀어내어 사용합니다.</li>
    </ul>
    <br>
    
    <h2 id="ch7">👻 Chapter 7 : 프록시와 연관관계 관리</h2>
    <h3>프록시 (Proxy)</h3>
    <ul>
        <li>엔티티를 조회할 때 연관된 엔티티까지 항상 함께 조회하는 것은 비효율적입니다.</li>
        <li>JPA는 실제 엔티티가 사용되는 시점까지 데이터베이스 조회를 미루기 위해 <strong>가짜 객체(프록시)</strong>를 사용합니다. 이것이 바로 <strong>지연 로딩</strong>의 핵심 메커니즘입니다.</li>
        <li>프록시 객체는 실제 클래스를 상속받아 만들어지며, 겉모양이 같아 개발자는 구분 없이 사용할 수 있습니다.</li>
        <li>프록시 객체의 메서드를 처음 호출할 때, JPA가 DB를 조회하여 실제 엔티티를 생성합니다. (프록시 초기화)</li>
    </ul>
    <h3>즉시 로딩 (Eager Loading) vs 지연 로딩 (Lazy Loading)</h3>
    <ul>
        <li><strong>즉시 로딩 (<code>FetchType.EAGER</code>)</strong>: 엔티티를 조회할 때 연관된 엔티티도 함께 조회합니다.</li>
        <li><strong>지연 로딩 (<code>FetchType.LAZY</code>)</strong>: 연관된 엔티티를 프록시로 조회하고, 실제 사용될 때 DB에서 가져옵니다.</li>
    </ul>
    <blockquote>⚠️ <strong>주의!</strong><br>
    실무에서는 <strong>모든 연관관계를 지연 로딩으로 설정</strong>하는 것을 원칙으로 합니다. 즉시 로딩은 JPQL 사용 시 N+1 문제를 일으키는 주요 원인입니다.
    </blockquote>
    <h3>영속성 전이 (CASCADE)</h3>
    <ul>
        <li>특정 엔티티를 영속화할 때 연관된 엔티티도 함께 영속화하는 기능입니다.</li>
        <li><code>cascade = CascadeType.ALL</code>, <code>CascadeType.PERSIST</code> 등의 옵션이 있습니다.</li>
        <li>두 엔티티의 생명주기가 거의 동일할 때 사용하면 편리합니다. (예: <code>Board</code>와 <code>Attachment</code>)</li>
    </ul>
    <h3>고아 객체 (Orphan Removal)</h3>
    <ul>
        <li>부모 엔티티와 연관관계가 끊어진 자식 엔티티를 자동으로 삭제하는 기능입니다.</li>
        <li><code>orphanRemoval = true</code> 옵션을 사용합니다.</li>
        <li>부모 컬렉션에서 자식을 제거하면, DB에서도 해당 데이터가 DELETE 됩니다.</li>
        <li><code>CascadeType.ALL + orphanRemoval = true</code> 두 옵션을 모두 사용하면, 부모 엔티티를 통해 자식의 생명주기를 완벽하게 관리할 수 있습니다.</li>
    </ul>
    <br>

    <h2 id="ch8">💎 Chapter 8 : 값 타입</h2>
    <h3>JPA의 데이터 타입 분류</h3>
    <ol>
        <li><strong>엔티티 타입 (<code>@Entity</code>)</strong>: 식별자(<code>@Id</code>)로 추적이 가능한 객체.</li>
        <li><strong>값 타입</strong>: 식별자 없이 값으로만 사용되는 타입. 생명주기를 엔티티에 의존합니다.
            <ul>
                <li><strong>기본값 타입</strong>: <code>int</code>, <code>String</code> 등 자바 기본 타입.</li>
                <li><strong>임베디드 타입 (복합 값 타입)</strong>: 사용자가 직접 정의한 값 타입. (예: <code>Address</code>)</li>
                <li><strong>컬렉션 값 타입</strong>: 값 타입을 하나 이상 저장할 때 사용.</li>
            </ul>
        </li>
    </ol>
    <h3>임베디드 타입 (Embedded Type)</h3>
    <ul>
        <li>새로운 값 타입을 직접 정의하여 재사용성과 응집도를 높일 수 있습니다.</li>
        <li><code>@Embeddable</code>: 값 타입을 정의하는 클래스에 명시합니다.</li>
        <li><code>@Embedded</code>: 값 타입을 사용하는 필드에 명시합니다.</li>
    </ul>
    <h3>값 타입과 불변 객체</h3>
    <ul>
        <li>임베디드 타입 같은 객체 값 타입은 여러 엔티티에서 공유될 경우, 한쪽의 변경이 다른 쪽에 영향을 미치는 <strong>부작용(Side Effect)</strong>이 발생할 수 있습니다.</li>
        <li>이러한 문제를 원천 차단하기 위해 값 타입은 <strong>불변 객체(Immutable Object)</strong>로 설계해야 합니다.
            <ul>
                <li>생성자로만 초기값을 설정하고, <strong><code>@Setter</code>를 만들지 않습니다.</strong></li>
            </ul>
        </li>
    </ul>
    <br>

    <h2 id="ch9">🔍 Chapter 9 : 객체지향 쿼리 언어 (JPQL)</h2>
    <h3>JPQL (Java Persistence Query Language)</h3>
    <ul>
        <li><strong>테이블이 아닌 엔티티 객체를 대상</strong>으로 질의하는 객체지향 쿼리입니다.</li>
        <li>SQL을 추상화하여 특정 데이터베이스 문법에 의존하지 않습니다.</li>
        <li>JPQL은 결국 SQL로 변환되어 실행됩니다.</li>
    </ul>
    <h3>Fetch Join (페치 조인)</h3>
    <ul>
        <li>JPQL에서 성능 최적화를 위해 제공하는 가장 중요한 기능입니다.</li>
        <li><strong>N+1 문제</strong>를 해결하는 효과적인 방법입니다.</li>
        <li>연관된 엔티티나 컬렉션을 SQL 한 번으로 함께 조회합니다.</li>
        <li>문법: <code>SELECT m FROM Member m JOIN FETCH m.team</code></li>
        <li>일반 JOIN과의 차이: 일반 JOIN은 연관된 엔티티를 함께 조회하지 않지만, <strong>페치 조인은 조회된 엔티티가 영속성 컨텍스트에 모두 로드되도록 보장</strong>합니다.</li>
    </ul>
    <blockquote>⚠️ <strong>페치 조인 사용 시 주의사항</strong>
        <ul>
            <li>둘 이상의 컬렉션을 페치 조인할 수 없습니다.</li>
            <li>컬렉션을 페치 조인하면 페이징 API(<code>Pageable</code>)를 사용할 수 없습니다.</li>
        </ul>
    </blockquote>
    <h3>벌크 연산</h3>
    <ul>
        <li>단일 쿼리로 여러 로우를 수정하거나 삭제하는 기능입니다.</li>
        <li><strong>영속성 컨텍스트를 거치지 않고 데이터베이스에 직접 쿼리를 실행</strong>하므로 주의가 필요합니다.</li>
        <li>벌크 연산 수행 후에는 영속성 컨텍스트와 DB의 데이터가 다를 수 있으므로, <strong>영속성 컨텍스트를 초기화(<code>em.clear()</code>)</strong> 하는 것이 안전합니다.</li>
    </ul>

</body>
</html>
