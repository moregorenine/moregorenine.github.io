---
title: "Spring Boot Quick Start #2"
excerpt: "Spring Boot에 대해 알아보자 JPA"
categories: 
  - springboot
tags: 
  - springboot
  - JPA
last_modified_at: 2018-11-07T22:00:00+09:00
toc: true
toc_sticky: true
---

## Intro
Spring Boot에 대해 알아보자 설정에서 부터  
[Spring Boot Quick Start 강좌 - Java Brains](https://javabrains.io/courses/spring_bootquickstart/ "Spring Boot Quick Start 강좌 Link")  
[연습예제 github](https://github.com/moregorenine/study/tree/master/spring-data-jpa "연습예제 github Link")

## JPA의 시작은?
JPA는 특정 DB에 종속되지 않는 범용적인 DB 접근방식이다.
Entity 객체를 클래스로 선언하고 해당 클래스가 테이블 이랑 매핑 된다고 보면 된다. 우리는 어떤 DB를 사용할지 알 필요가 없다. 해당 Entity 객체에 대한 CRUD를 작성하면 차후 연결되는 DB에 JPA library가 알아서 쿼리를 작성해 mapping해 주다고 보면 된다. 이는 DB에 종속되지 않기에 DB변경으로 부터 자유롭고 빠른 개발을 가능케하고 쿼리 오류를 줄여준다. 만약 당신이 '어 내가 작성하는 쿼리보다 성능이 낮으면 어떻하지?'라고 걱정할수도 있겠지만... JPA 제공하는 업체에서는 자신들의 쿼리 자동 변경에 대해 상당한 자부심을 느끼고 있으니 천재가 아니라면 그냥 넘어가자.  
우선 이번 화는 Web, JPA, Apach Derby 만 최소화 하여 선택 진행한다.  
[![foo](https://c1.staticflickr.com/5/4804/30825041407_6cc3be911d_o.png)](https://flic.kr/p/NXUoTa)  

## Entity 객체
우선 테이블과 매핑될 Entity 객체를 생성한다. Topic이란 객체는 id를 PK로 가지며 name, description 컬럼을 가지는 테이블이 될 것이다.  
- Topic.java
{% highlight java linenos %}
import javax.persistence.Entity;
import javax.persistence.Id;

@Entity
public class Topic {
	@Id
	String id;
	String name;
	String description;

	public Topic() {
		super();
	}

	public Topic(String id, String name, String description) {
		super();
		this.id = id;
		this.name = name;
		this.description = description;
	}

	public String getId() {
		return id;
	}

	public void setId(String id) {
		this.id = id;
	}

	public String getName() {
		return name;
	}

	public void setName(String name) {
		this.name = name;
	}

	public String getDescription() {
		return description;
	}

	public void setDescription(String description) {
		this.description = description;
	}

}
{% endhighlight %}

## controller 객체
httprequest 요청 주소를 @RequestMapping에 선언한다. @RequestMapping("/topics/{id}")에 {id} 부분은 요청 url 주소에 해당 영역에 해당하는 정보를 @PathVariable String id 에서 받는다는 뜻이다. Rest API에서 권고하는 url 매핑 정보이다. method=RequestMethod.XXX에서 request에서 사용할 방식을 선언한다.  
- TopicController.java
{% highlight java linenos %}
@RestController
public class TopicController {

	@Autowired
	TopicService topicService;

	@RequestMapping("/topics")
	public List<Topic> getAllTopics() {
		return topicService.getAllTopics();
	}
	
	@RequestMapping("/topics/{id}")
	public Topic getTopic(@PathVariable String id) {
		return topicService.getTopic(id);
	}
	
	@RequestMapping(method=RequestMethod.POST, value="/topics")
	public void addTopic(@RequestBody Topic topic) {
		topicService.addTopic(topic);
	}
	
	@RequestMapping(method=RequestMethod.PUT, value="/topics/{id}")
	public void updateTopic(@RequestBody Topic topic, @PathVariable String id) {
		topicService.updateTopic(id, topic);
	}
	
	@RequestMapping(method=RequestMethod.DELETE, value="/topics/{id}")
	public void deleteTopic(@PathVariable String id) {
		topicService.deleteTopic(id);
	}
}
{% endhighlight %}

## service 객체
- TopicService.java
repository를 이용해 실질적으로 crud를 실행한다.  
{% highlight java linenos %}
@Service
public class TopicService {

	@Autowired
	private TopicRepository topicRepository;

	public List<Topic> getAllTopics() {
		List<Topic> topics = new ArrayList<>();
		topicRepository.findAll().forEach(topics::add);
		return topics;
	}

	public Topic getTopic(String id) {
		return topicRepository.findById(id).orElse(null);
	}

	public void addTopic(Topic topic) {
		topicRepository.save(topic);
	}

	public void updateTopic(String id, Topic topic) {
		topicRepository.save(topic);
	}

	public void deleteTopic(String id) {
		topicRepository.deleteById(id);

	}
}
{% endhighlight %}

## Repository 객체
- TopicRepository.java
아무런 내용이 없다. 왜냐하면 기본적으로 CrudRepository에서 제공하는 JPA 기능들을 그대로 사용할 것이기 떄문이다. 다만 CrudRepository<Topic, String> 부분에 해당 repository에서 사용하는 객체와 해당 객체의 id 클래스를 입력해준다.  
{% highlight java linenos %}
@Service
public interface TopicRepository extends CrudRepository<Topic, String> {
}
{% endhighlight %}

## Related Posts
Spring Boot Quick Start #1 [link](https://moregorenine.github.io/springboot/springboot-1/ "Spring Boot Quick Start #1")