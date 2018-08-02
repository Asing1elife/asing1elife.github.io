---
title: Hibernate 双向关联转换 JSON 防止死循环
date: 2018-05-17
author: asing1elife
categories:
 - java
 - hibernate
tags:
 - java
 - hibernate
---
> Hibernate 中经常存在双向关联的情况，此处将数据转换为 JSON 格式则可能导致死循环  

## 获取数据时只需要单向关联
1. 这种情况下父类需要子类数据，而子类不需要父类数据

```java
@Entity
@Table(name = "ht_feedback_template")
public class FeedbackTemplate extends BaseModel {
    @OneToMany(mappedBy = "feedbackTemplate", cascade = {CascadeType.ALL}, fetch = FetchType.LAZY)
    @OrderBy("indexNo asc")
    private Set<FeedbackQuestion> questions;

    public Set<FeedbackQuestion> getQuestions() {
        return questions;
    }

    public void setQuestions(Set<FeedbackQuestion> questions) {
        this.questions = questions;
    }
}
```

2. 在子类中对父类的引用字段加上 `@JsonIgnore` 即可

```java
@Entity
@Table(name = "ht_feedback_question")
public class FeedbackQuestion extends BaseModel {
    @ManyToOne
    @JsonIgnore
    private FeedbackTemplate feedbackTemplate;

    public FeedbackTemplate getFeedbackTemplate() {
        return feedbackTemplate;
    }

    public void setFeedbackTemplate(FeedbackTemplate feedbackTemplate) {
        this.feedbackTemplate = feedbackTemplate;
    }
}
```

## 获取数据时确实需要双向关联
1. 在父类对子类的引用字段加上 `@JsonManagedReference`

```java
@Entity
@Table(name = "ht_feedback_template")
public class FeedbackTemplate extends BaseModel {
    @OneToMany(mappedBy = "feedbackTemplate", cascade = {CascadeType.ALL}, fetch = FetchType.LAZY)
    @OrderBy("indexNo asc")
  @JsonManagedReference
    private Set<FeedbackQuestion> questions;

    public Set<FeedbackQuestion> getQuestions() {
        return questions;
    }

    public void setQuestions(Set<FeedbackQuestion> questions) {
        this.questions = questions;
    }
}
```

2. 在子类中对父类的引用字段加上 `@JsonBackReference` 即可

```java
@Entity
@Table(name = "ht_feedback_question")
public class FeedbackQuestion extends BaseModel {
    @ManyToOne
  @JsonBackReference
    private FeedbackTemplate feedbackTemplate;

    public FeedbackTemplate getFeedbackTemplate() {
        return feedbackTemplate;
    }

    public void setFeedbackTemplate(FeedbackTemplate feedbackTemplate) {
        this.feedbackTemplate = feedbackTemplate;
    }
}
```