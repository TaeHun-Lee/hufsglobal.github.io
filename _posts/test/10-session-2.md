---
layout: post
title: 5. UD복습, form, JS팝업
category: session
---
세션 10 - prod. 김은진


# 10th session 

## Contents
1. UD 복습
  - update
  - delete
2. 폼(form) 
3. JS팝업

---
***
___

## 1. UD 복습

###복습





---
***
___


## 2. form 

  * Form : HTML에서 웹 페이지상에서 한 개 이상의 필드나 위젯들의 묶음을 말하며, 
  사용자로부터 정보를 수집하여 서버에 제출하는데 이용된다.

  - 기존에는 model에서 title, content, date 등을 정의해주고 그 속성들 그대로 html을 제작한다.
  - model 기반/맞춤형 form을 통해 이전의 귀찮은 작업들을 쉽게 편리하게 이용/저장가능해진다.

  - form은 django의 가장 큰 feature 중 하나로 model 클래스와 유사하게 form class를 정의한다.
  - 일반적인 폼(form)은 직접 필드를 정의하거나 위젯 설정이 필요한데, 
  모델폼(model form)은 모델과 필드를 지정하면 모델폼이 자동으로 폼필드를 생성한다.

  - ex) form과 model form 비교
  {% highlight html %}
  {% raw %}
  #In model.py

  from django import forms
  from .models import Post

  #Form (일반 폼)
  class PostForm(forms.Form):
    title = forms.CharField()
    content = forms.CharField(widget=forms.Textarea)

  #Model Form (모델 폼)
  class PostForm(forms.ModelForm):
    class Meta:
      model = Post
      fields = ['title', 'content']
  {% endraw %}
  {% endhighlight %}
    
1. forms.py 생성

  - models.py파일과 동일한 경로에 forms.py파일 생성 
  -> views.py에서 import해와서 사용할 것이다. (효율적인 코드관리를 위해 파일을 나눈다.)

  - model 맞춤형 form 생성해주기

  {% highlight html %}
  {% raw %}

  from django import forms
  from .models import Blog

  class BlogPost(forms.ModelForm):
          class Meta:
              model = Blog
              fields = ['title','body']

  {% endraw %}
  {% endhighlight %}
  - 메타클래스를 선언하고 model, fields를 정의해준다.
  - class Meta: : 클래스를 만드는데 사용되는 것으로, 메타클래스의 인스턴스가 클래스라는 것이다.
  - model = Blog : 'Blog'라는 모델을 기반으로 form을 만들 것인가 정의
  - fields = [] : 모델의 속성 중엥서 입력받길 원하는 속성 정의


2. views.py 수정

  * 서버로 데이터를 전송하는 방식
  - POST방식 : 보이지 않는 타입으로, form에서 받은 데이터들을 url에 직접 보여주지 않고 데이터들을 저장해서 서버로 보낸다.
  - GET방식 : url에 데이터가 그대로 보이는 타입으로, 우리가 만든 form을 보여주면 된다.

  {% highlight html %}
  {% raw %}
  #forms.py의 BlogPost라는 class를 가져와서 사용
  from .forms import BlogPost

  def blogpost(request):

  #request의 method가 post방식인 경우, 새로운 blog를 post한다는 뜻이므로 사용자가 form에 입력한 데이터를 저장한다는 뜻
      if request.method =='POST':
          form = BlogPost(request.POST)
          if form.is_valid():
              post = form.save(commit=False)
              post.pub_date=timezone.now()
              post.save()
              return redirect('home')
  #request가 get방식인 경우, 사용자가 입력하려고 blogpost(새로만들기)을 누른 것이므로 입력받으려는 form을 보여줘라는 뜻
      else:
          form = BlogPost()
          return render(request,'new.html',{'form':form})
  {% endraw %}
  {% endhighlight %}

  * post인 경우
  - form = BlogPost(request.POST) : BlogPost 객체를 만들어 받아온 데이터를 입력한다.
  - is_valid() : 입력값들이 유효한지 판단하는 함수로, 유효하다면 commit=False를 통해 저장하지 않고 데이터만 가져온다.
  - 저장하지 않고 가져오는 이유는 pub_date를 입력받지 않았기 때문이다.
  - pub_date 라는 속성에 timezone.now()은 현재 시각이 들어가도록 사용자가 직접 입력하지 않아도 자동으로 blog을 쓸 때 입력되도록 한다.

  * get인 경우
  - BlogPost 객체를 만들어 받아온 데이터를 입력한다.
  - 요청이 들어오면 new.html을 띄우는데 form은 dictionary 자료형으로 넣어서 전달한다.

3. urls.py 수정

  {% highlight html %}
  {% raw %}

  #blog/newblog로 접속하면 views.py의 blogpost라는 함수가 실행되도록 path 설정
  path('newblog/', views.blogpost, name='newblog')

  {% endraw %}
  {% endhighlight %}

4. templates 수정

  {% highlight html %}
  {% raw %}

  {% extends 'base.html' %}
  {% block content %}
  <br>

  <div class='container'>
      <form method='POST'>
          {% csrf_token %}
          <table>
              {{form.as_table}}
          </table>
          <br>
          <input class="btn btn-dark" type="submit" value="제출하기">
      </form>
  </div>
  {%endblock%}


  {% endraw %}
  {% endhighlight %}

  * 템플릿 상속(block content를 통해) 입력 페이지 html 완성
  * as_table : form을 table형식으로 출력하는 form의 method(이외에도 as_p, as_ul 등이 있다.)
  * input 태그를 통해 submit버튼을 클릭하면 작성했던 form이 views.py로 이동되어 처리된다.


* 자세한 순서는 ppt를 참조하세요.
<>
---
***
___


## 3. JS 팝업

* 팝업은 회원가입이나 아이디 중복검색, 우편번호 검색등을 사용할 때 주로 사용
* popup.html에서 팝업창 호출하는 페이지(부모창), popuptest.html에서 팝업창 페이지(자식창), jquery.html은 팝업창에서 이동하는 페이지

1. 팝업 호출 글(부모창) 작성

2. input 태그 작성 

{% highlight html %}
{% raw %}

<input type="button" value="팝업창 호출" onclick="showPopup();" />

{% endraw %}
{% endhighlight %}

- 클릭할 때 무언가 튀어나와야(pop up)하는 input태그 안에 onclick을 통해
시도하고자 하는 java script의 function의 이름을 적는다.

3. script 작성

{% highlight html %}
{% raw %}

<script>
function showPopup() { window.open("popuptest.html", "popup1", "width=400, height=300, left=100, top=50"); }
</script>

{% endraw %}
{% endhighlight %}

- script는 onclick을 통해 시행되는 java script로 소괄호를 호출하는 메소드다.
- window.open : 화면을 띄우는데 팝업창으로 열릴 html을 적고,
popup1은 이 팝업창이 어떤 것을 의미하는지 알아보기 쉽게 적고,
그 외는 크기와 위치 등을 정할 수 있다.


4. popuptest.html(팝업창)
- 이 html이 팝업창이라는 것을 보여주기 위해 <h1>popup</h1>을 적는다
- 정밀하게 팝업창의 크기를 조절하려면 바디태그에 body onload = "window.resizeTo(300,300)" 과 같이 조절 가능

<input type = "button" value = "닫기" onclick="self.close();" /> 
<input type = "button" value = "이동 후 닫기" onclick="moveClose();" />

- 닫기 버튼 : input 태그에 onclick= "self.close();"를 통해 self 자기자신 창을 닫는 메소드를 추가
- 이동 후 닫기 버튼 : input 태그에 onclick 속성을 넣어 이동 후 닫히는 onclick="moveClose();를 통해 이동 후 닫히는 메소드 추가
- self.close()는 이미 정의된 java script(js) 메소드고 moveClose는 우리가 만들 것이기 때문에 밑에 script를 열어 새로 정의를 한다.

{% highlight html %}
{% raw %}

<script>
    function moveClose(){
        opener.location.href="jquery.html";
        self.close();
    }
</script>

{% endraw %}
{% endhighlight %}

- opener.location.href : 원하는 해당 url로 가기 위해 사용하는 방법으로 opener는 부모창과 자식창을 가진다.
- location : 브라우저의 창의 열려있는 문서의 url을 알려주는 객체로 opner 없이는 부모페이지가 액션을 받지 못한다.
- 부모창 : 처음 버튼을 클릭하는 페이지로 자식창의 정보를 가지고 있다.
- 자식창 : 팝업창(눌러서 새로 생성되는 페이지)으로 부모창의 정보를 가지고 있다.

- script에 js를 만들 수도 있고 <head>에 넣어도 되고, js파일을 만들어 <head>에 링크를 연결해서 사용도 가능

5. result.html(이동될 페이지)
- 이동 후 닫기 버튼을 누른 뒤에 뜨는 페이지

* 자세한 순서는 ppt를 참조하세요.
<>
---
***
___
