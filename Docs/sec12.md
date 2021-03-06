# section 12

# 12.0 URLs and Arguments (11:06)

```python
#config - urls
urlpatterns = [
    path("", include("core.urls", namespace="core")),
    path("rooms/", include("rooms.urls", namespace="rooms")), #rooms/로 시작하면 rooms -> urls로 라우팅
    path("admin/", admin.site.urls),
]

```

```python
#core - urls
from django.urls import path
from rooms import views as room_views
app_name = "core"
urlpatterns = [path("", room_views.HomeView.as_view(), name="home")]
```

- url구성시 : ?page = 1 쿼리를 주는경우 | 지금은 rooms/4, rooms/15 등등의 url를 받을때 사용함.

```python
#rooms - urls
from django.urls import path
from . import views
app_name = "rooms"
urlpatterns = [path("<int:pk>", views.room_detail, name="detail")] #
```

- 반드시 request와 pk도 받아야 한다.

```python
def room_detail(request, pk):
    print(pk)
    return render(request, "rooms/room_detail.html")
```

- href에 다음처럼, url 길을 직접 지정해 줘도 되지만, include의 namespace 와 path 의 name으로 namespace:name으로 url를 연결할수도 있음.!!

```
#room_list.html
    <a href ="rooms/{{room.pk}}"> <h4> {{room.name }} / {{room.price}} </h4> </a>
```

```
#room_list.html
    <a href ="{% url "rooms:detail" room.pk %}"> <h4> {{room.name }} / {{room.price}} </h4> </a>

```

```
#header.html
    <header>
      <a href="{% url "core:home" %}">Nbnb</a>
      <ul>
        <li><a href="#">Login</a></li>
      </ul>
    </header>
```

# 12.1 get_absolute_url (4:07)

- rooms의 하나의 인스턴스로 admin에 들어갔을떄 view on site라는 버튼을 통해서 detail 프론트 페이지로 연결할수있다.
- from django.urls import reverse === {% url "rooms:detail" %} | reverse( namespace:name , kwargs..)

```
#rooms - model의 overide되는 함수 : get_absolute_url
    def get_absolute_url(self):
        return reverse("rooms:detail", kwargs={"pk": self.pk})
```

# 12.2 room_detail FBV finished (8:03)

- http://localhost:8000/rooms/117 에서 없는 방번호로 url접근하면 DoesNotExist 애러 발생 -> core:home으로 리버스 리다이렉트 시킨다.  
  [참고 문서](https://docs.djangoproject.com/en/3.0/ref/models/instances/#doesnotexist)

### room - view

```python
def room_detail(request, pk):
    try:
        room = models.Room.objects.get(pk=pk) #pk를 통해서 방을 얻어옴
        return render(request, "rooms/detail.html", context={"room": room}) #랜더
    except models.Room.DoesNotExist: #예외 - 없는pk를 요청시
        return redirect(reverse("core:home"))
```

### room_detail.html

```html
{% extends "base.html" %} {% block page_name %} Detail {% endblock page_name %}
{% block content %}
<div>
  <h1>{{room.name}}</h1>
  <h3>{{room.description}}</h3>
</div>
<div>
  <h2>
    By: {{room.host.username}} {% if room.host.superhost %} (superhost) {% endif
    %}
  </h2>
  <h3>Amenities</h3>
  <ul>
    {% for a in room.amenity.all %}
    <li>{{a}}</li>
    {% endfor %}
  </ul>
  ...
</div>
{% endblock content %}
```

# 12.3 Http404() (4:33)

- 디버그 모드는 False, allowed host는 "\*"로 설정 -> 다 접속 가능하게 한다.~
- 서버애러는 500 , NOT Found는 404 이다.
- url난동을 부리면 404d애러를 raise 해준다. 만약 처리를하지않으면 DoesNotExist 예외가 발생하면: 서버애러이다. 즉 500이다.

- ex)존재하지 않는page를 요청할때 애러

```python
from django.http import Http404
...
    except models.Room.DoesNotExist:
        raise Http404()
```

- 반드시 404.html이름으로 templates폴더에 있어야 한다. 자동으로 다음의 404.html을 띄어준다.

```python
{% extends "base.html" %}

{% block page_name %}
  ERROR
{% endblock page_name %}

{% block content %}
    <h1> NOT FOUND !! </h1>
{% endblock content %}
```

```
DEBUG = False

ALLOWED_HOSTS = "*"
```

# 12.4 Using DetailView CBV (6:33)

- when? 하나의 모델인스턴스만 가지고 페이지를 만들때! eg) room상세정보
- detail뷰 구조 : pk(urls) -> view전달 -> 하나의 object만 가지고 -> 템플릿 자동연결. room_detail | 모델이름+detail(view제너릭이름)
- detail뷰에서 model은 -> 템플릿으로 자동전달 : room | object 로 자동 명명됨.
- detail뷰 사용시: 없는 pk시 알아서 404 not found처리해준다.

### http://localhost:8000/rooms/52 요청시 pk = 52

### rooms - urls

```python
urlpatterns = [path("<int:pk>", views.RoomDetail.as_view(), name="detail")] #pk는 장룰
```

### rooms - views

```python
class RoomDetail(DetailView):
    model = models.Room #쿼리셋 지정
    pk_url_kwarg = "pk" #자동으로 pk라는 장룰변수로 쿼리를 얻어온다.(<int:pk> 변수명이 일치한다면 생략가능)
```

### templates - room_detail.html (명명 장룰)

```html
<h1>{{object.name}}</h1>
<h3>{{room.description}}</h3>
room.host.username room.amenity.all...으로 페이지 꾸미면됨.
```
