# Spring MVC

- Model, View 그리고 Controller로 분리한 Design pattern

> 해당 포스트에 작성된 코드는 실제 정의된 코드가 아니라 개념을 익히기 위해 유사하게 구현했습니다.

<br>

![png](/_img/210629_simpleRequest.png)

- Client의 요청을 처리하는 business 로직과 view 렌더링을 1개의 파일 안에서 구현할 수 있다.
- Business Logic 수정시 view 코드를 변경할 필요가 없고 반대의 경우도 마찬가지다. 
- 즉, 개발 시 **라이프 사이클이 다른 경우엔 분리해서 작성해야 유지 보수가 좋아진다**. 
- 이를 위해 MVC 패턴이 만들어졌다.
<br>

- [Servlet](#servlet-으로-controller-와-view-역할-구분)
- [DispatcherServlet](#dispatcherservlet)
- [View 완전히 분리](#view-분리)
- [ViewResolver](#viewResolver)
- [return View의 논리 주소](#return-view)
- [다양한 방법을 처리하는 HandlerAdapter](#handlerAdapter)

<br>

# Servlet 으로 controller 와 view 역할 구분

![png](/_img/210628_mvc1.png)

1. Client가 요청하면 **Controller**는 HTTP 요청을 받아 파라미터를 검증한다.
2. Controller는 **Business Logic**을 실행시켜 데이터를 처리한다.
3. Business Logic을 통해 처리된 데이터를 **View**에 전달하기 위해 **Model에 저장**한다. 이때 Model은 ```HttpServletRequest``` 객체이고 내부에 데이터 저장소를 갖고있다.
4. Controller는 View을 호출하면 해당 view는 Model에 담겨있는 데이터를 사용해 노출할 화면을 HTML로 생성한다. (view는 Model의 주소값을 넘겨 받는다.)
5. Client 요청에 대한 응답을 한다.
<br>

![png](/_img/210628_mvcMemberSaveServletCode.png)

> Commit: https://github.com/evelyn82ny/Model-View-Controller-pattern/commit/adc971694cbfde64daa9cf05584bc48e0b0645fa

회원을 저장하는 과정을 MVC 패턴에 적용하면 위와 같은 코드가 된다.

- 요청받은 파라미터를 검증 및 파싱하고
- business 로직을 수행한 뒤
- Model에 데이터를 저장하고
- view 로직를 통해 화면을 노출한다.

<br>

## HttpServlet

```java
public abstract class HttpServlet extends GenericServlet {
    protected void doGet(HttpServletRequest req, HttpServletResponse resp)
            throws ServletException, IOException {...}

    protected void doPost(HttpServletRequest req, HttpServletResponse resp)
            throws ServletException, IOException {...}
}
```
- HttpServlet 추상 클래스를 상속한다.
- HttpServlet 의 service method를 ```@Override``` 하여 원하는 로직을 구현한다.
<br>

- Client가 요청하면 Servlet의 ```Service()```가 호출되고
- ```Service()```는 GET/POST 를 구분해 ```doGet()/doPost()```를 호출한다.
- 두 method는 ```HttpServletRequest```와 ```HttpServletResponse```를 parameter로 가지고 있는데 Servlet과 Client 사이를 연결한다.
<br>

## @WebServlet

- 위 MvcMemberSaveServlet 은 member의 정보를 저장하기 위한 Servlet이다. 
- 즉, 여러 기능에 대한 여러 Servlet을 생성할 수 있다.
- 그러면 특정 기능을 수행하는 Servlet이 무엇인지 파악해야 하는데 이를 ```@WebServlet``` 이 수행한다.
- urlPatterns에 작성한 ```/servlet-mvc/members/save``` url로 요청이 들어오면 ServletContainer는 MvcMemberSaveServlet을 호출한다.
<br>

## Model

- 데이터를 저장 및 조회하는 model의 역할을 ```HttpServletRequest``` 가 담당한다.
- ```HttpServletRequest``` 내부에 데이터 저장소가 있고
- ```setAttribute()```, ```getAttribute()``` 으로 데이터 보관 및 조회한다.
<br>

## WEB-INF

- Business logic을 처리한 다음 view를 호출한다.
- view 경로를 보면 **WEB-INF** 에서 시작한다.
- WEB-INF 경로안에 있는 html 을 외부에서 직접 호출할 수 없다.
- 그러므로 **WEB-INF 경로안에 있는 view 는 Controller 에 의해서만 실행**될 수 있다.

<br>

## Forward vs Redirect

### Forward

- forward(request, response) 로 다른 servlet 이나 view 로 이동한다. 
- **Forward 는 서버 내부에서 일어나는 호출이므로 Client가 변경을 인지하지 못한다**.
<br>

### Redirect

- Redirect는 client에게 응답을 보내고 client가 redirect 경로를 요청한다.
- 그러므로 **Client가 변경을 인지하고 실제 url도 변경**된다.

<br>

# DispatcherServlet

```Servlet```을 사용해 **Controller의 역할과 View를 Rendering 하는 역할을 명확하게 구분**했다.<br>
회원을 저장, 조회하는 등 많은 기능을 위 코드와 같이 ```HttpServlet```을 상속해 ```Service()```을 ```@Override```하여 작성할 수 있지만 다음과 같은 문제가 발생한다.<br>

- View 경로가 모두 비슷한 경로이거나 View 경로가 바뀌거나 다른 View로 변경한다면 모든 코드를 수정해야한다.
- ```RequestDispatcher.forward()```도 모든 코드에서 사용되니 중복된다.
- 서비스 규모가 커질수록 공통으로 처리해야하는 부분이 많아지기 때문에 공통 기능을 처리하는 것이 필요하다.

Front Controller 을 구현해 위 문제를 해결해보고 Front Controller는 실제 Spring의 ```DispatcherServlet```과 같다.
<br>

![png](/_img/front_controller.png)

- FrontControllerServlet 1개로 모든 클라이언트 요청을 받는다. 
- FrontController는 요청에 맞는 Controller를 호출한다. 
- FrontController는 공통 처리하는 servlet이므로 controller에선 더이상 servlet을 사용하지 않아도 된다. 
- 즉, controller에서 ```@WebServlet``` 으로 url 을 mapping 하거나 ```HttpServlet```을 상속받지 않아도 된다.
<br>

![png](/_img/front_controller_mapping.png)

- HTTP request에 맞는 controller를 찾기 위해 FrontController는 mapping 정보에서 controller를 찾는다.
- FrontController는 url과 mapping되는 controller를 호출하고 JSP forward하여 html 응답한다.
- 실제 spring MVC 의 ```DispatcherServlet```이 위 과정으로 동작한다.
<br>

## HandlerMapping (Controller URL Mapping)

- 여러 Controller 중 특정 Controller를 찾기 위해 Mapping 정보가 필요했다.
- 위 Mapping 정보가 실제 Spring의 ```HandlerMapping``` 이다.
- Client 요청을 처리할 Controller를 찾는다.
- 요청 URL에 해당하는 Controller 정보를 저장하는 Table이 존재한다.

<br>

다형성을 이용하기 위해 controller를 Interface로 구현하고 회원 저장, 조회 등 각각의 기능은 controller interface 를 상속받아 구현했다.
여러 Controller를 구분하는 FrontControllerServlet 코드는 다음과 같다.

![png](/_img/210628_frontControllerServlet_code.png)

> Commit: https://github.com/evelyn82ny/Model-View-Controller-pattern/commit/a66abb5b9977a1ed2fd867f2302580ba9a27a4ea

<br>

## 모든 요청 Front Controller로 받기

- ```@WebServlet``` 의 urlPattern 을 ```/front-controller/*``` 로 작성하면 ```/front-controller``` 하위 url은 해당 servlet으로 들어온다.
<br>

## HttpServletRequest.getRequestURI()

- controllerMap 에 **{ key: mapping URI , value: 호출될 controller }** 로 여러 controller를 저장했다. 
- ```getRequestURI()```로 요청된 URI를 읽고 controllerMap에서 mapping된 controller를 찾는다.
- controller를 찾았다면 해당 controller의 process를 수행한다.
- 만약 controller를 찾지 못했다면 404 상태 코드(SC_NOT_FOUND)를 반환한다.
<br>

# View 분리

위 과정을 통해 요청에 맞는 controller를 찾고 공통 부분 처리를 위한 ```FrontControllerServlet```를 추가했다.<br>
하지만 지금은 controller에서 view를 직접 Forward하는 상태이므로 controller와 view가 완전히 분리된 상태가 아니다.

![png](/_img/architecture_that_call_render_method.png)

- Controller에서 해당되는 View 정보를 FrontController로 반환한다. 
- 해당 Controller에서 반환된 View를 호출하면 Controller와 View가 완전히 분리된다.
- 여기서 말하는 View는 spring MVC의 View이다.
<br>

> Commit: https://github.com/evelyn82ny/Model-View-Controller-pattern/commit/0961c4eaeeea88a42bfbcb984206dbe6484a3c11

![png](/_img/210628_view_code.png)

![png](/_img/210628_view_frontControllerServlet.png)

- 모든 Controller는 해당되는 View 객체를 생성하여 ```FrontControllerServlet```에 반환한다. 
- ```FrontControllerServlet```은 받은 View의 ```render()```을 호출한다.
- 즉, ```FrontControllerServlet```를 통해 View 객체의 ```render()``` 호출을 일관되게 처리할 수 있게 되었다.
<br>

# ViewResolver

## 중복되는 View 경로 제거

- 모든 Controller 에서 View 경로를  ```return new View("/WEB-INF/views/save-result.jsp");```처럼 물리 경로를 반환한다. 
- **/WEB-INF/views** 는 중복되는 부분이고 만약 수많은 view 코드가 있는 views 폴더를 다른 폴더로 옮긴다면 모든 Controller 코드를 수정해야 한다.
- 이를 해결하기 위해 Controller는 해당되는 View의 논리 이름만 반환하고 FrontController는 받은 View 논리 이름을 물리 주소로 변경하면 중복을 처리할 수 있다.
<br>

- View 물리 주소 : /WEB-INF/views/save-result.jsp
- View 논리 이름 : save-result

<br>

## Servlet 종속성 제거

- 여러 Controller는 ```httpServletRequest```, ```httpServletResponse``` 를 파라미터로 받지만 모든 기능을 사용하지 않았다.
- 또한, Controller는 Servlet 기술이 아닌 처리 해야하는 데이터만 필요하므로 Servlet 자체를 넘겨주는게 불필요해 보인다.
- 이를 해결하기 위해 ```HttpServlet``` 객체를 Model로 사용하지 않고 Model 객체를 따로 생성한다.
- 즉, Controller가 servlet 기술없이 데이터만 처리할 수 있도록 servlet 종속성을 제거한다.
<br>

![png](/_img/architecture_with_view_resolver_applied.png)

> Commit: https://github.com/evelyn82ny/Model-View-Controller-pattern/commit/a4afa239426fdcb0c2ca0e7ac3c4778daf2afb2e

![png](/_img/210629_modelAndView_code.png)

- Controller는 FrontController에게 {View 논리 이름, 데이터가 담긴 Model}인 **ModelAndView** 객체를 반환한다.
- ModelAndView 코드를 보면 String으로 **View의 논리 이름**을, Map<String, Object>로 **Model**를 만들어 데이터를 담는다.
<br>

![png](/_img/210629_modelAndView_controllerCode.png)

- 위 코드는 회원을 저장하는 Controller이며 ```httpServlet``` 이 아닌 요청을 처리하기 위한 필요한 데이터만 파라미터로 전달받는다.
- 받은 데이터로 Business logic을 수행한 뒤 결과를 반환하기 위한 ModelAndView 객체를 생성한다.
- ModelAndView 객체의 View에는 결과를 출력할 **View 논리이름**을, Model에는 데이터 이름과 결과 데이터를 저장 후 FrontController에게 반환한다.
- View 논리 이름만 반환하기 때문에 View 경로가 바뀌어도 Controller를 수정할 필요가 없다.
<br>

![png](/_img/210629_viewResolver_code.png)

이제 ```FrontControllerServlet```은 ```ModelAndView``` 객체를 반환받고 ```ViewResolver```를 사용해 물리 주소로 바꾼다.

### createParamMap

- ```FrontControllerServlet```은 클라이언트의 요청에 따른 HTTP request message를 처리하여 ```HttpServletRequest```, ```HttpServletResponse``` 객체를 전달받는다. 
- 전달받은 HttpServlet을 그대로 Controller에게 전달하지 않고 처리해야하는 데이터만 전달한다.
- createParamMap 메소드로 ```HttpServletRequest```에서 데이터만 Map<String, String>에 저장한 뒤 controller에게 전달한다. 
- controller는 전달받은 데이터를 처리해 ModelAndView 객체로 반환한다.

### viewResolver

- Front Controller는 특정 Controller에게 받은 View 논리 이름을 물리 주소로 변경해 View 객체를 만든다.
- 만약 View 경로가 변경되면 이전처럼 특정 Controller 코드를 수정할 필요없이 새로운 주소 View 객체를 생성하면 된다.
- view의 ```render()```를 호출할 때 Model(데이터 정보)도 같이 넘겨 준다.

![png](/_img/210629_viewRender.png)

> Commit: https://github.com/evelyn82ny/Model-View-Controller-pattern/commit/a4afa239426fdcb0c2ca0e7ac3c4778daf2afb2e

- FrontControllerServlet에게 Model을 파라미터로 전달받는다.
- ```HttpServletRequest```와 ```HttpServletResponse``` 객체를 forward 하기 위해 ```setAttribute()```로 Model data를 ```HttpServletRequest``` 객체에 저장한다.
<br>
- ServletResponse의 ```getRequestDispatcher(the pathname to the resource)``` 를 호출해 지정된 경로에 있는 resource의 wrapper 역할하는 ```RequestDispatcher``` 객체를 얻는다.
- 그 후 RequestDispatcher의 ```forward(ServletRequest, ServletResponse)``` 를 호출해 Servlet에서 Server의 다른 리소스(JSP, HTML 등)로 요청을 전달한다.
<br>

# return View

- Model 객체를 통해 Servlet 종속성을 제거했고 ViewResolver 로 view 경로 작성이 중복되는 것을 해결해 자신의 역할에만 충실할 수 있는 설계가 되었다. 
- 하지만 FrontController가 아닌 특정 Controller에서 ModelAndView 객체를 생성하고 저장 후 반환해야하는 번거로움을 존재한다. 
- 이전보다 더 편리해지기 위해 Model 객체를 FrontController에서 만들고 Controller에게 Model 객체를 파라미터로 전달한다.

![png](/_img/architecture_in_which_the_view_is_returned_from_controller.png)

- FrontController Servlet 에서 Map<String, Object> model 객체를 만든다. 
- 실행해야 할 controller 에게 model 객체를 파라미터로 넘겨주고 Controller에게 View 논리이름을 반환받는다.

> Commit: https://github.com/evelyn82ny/Model-View-Controller-pattern/commit/7ac12856bdb8fdd44459efb907278382133ddba3

![png](/_img/210629_returnModel_code.png)

- rendering을 위해 View의 ```render()``` 호출 시 FrontControllerServlet에 있는 model을 그대로 넘겨준다.
- Controller는 파라미터로 전달받은 Model에 데이터를 저장한다. 
- 번거롭게 ModelAndView 객체를 생성할 필요없이 **View 논리 이름만 반환**하면 된다.
<br>

![png](/_img/210629_controller_parameterModel_code.png)

- FrontControllerServlet에서 model을 파라미터로 넘기고 View의 논리 이름만 반환하는 방식으로 변경했다.

<br>

# HandlerAdapter

어떤 부분은 ModelAndView 객체를 반환하는 Interface 기반의 controller를 사용하고 싶고, 다른 부분은 View 만 반환하는 interface 기반의 controller 를 사용하고 싶을 때 Adapter 개념을 적용한다.

- controller 명칭을 Handler로 변경한다. 
- Adapter로 어떤 형태든 상관없이 모두 처리가능하기 때문에 controller 보다 더 넓은 범위인 Handler로 명칭을 변경한다.

![png](/_img/architecture_with_handler_adapter_applied.png)

1. HTTP 요청에 맞는 Handler (controller)를 찾는다.
2. Handler를 처리할 수 있는 Handler Adapter를 찾는다.
3. 반환받고자하는 객체를 handler adapter 에게 요구한다.
4. Handler Adapter 는 handler 를 호출해 요청사항을 수행한다.
5. Handler로 수행된 결과를 Front Controller가 요구하는 객체로 생성하여 반환한다.
6. ViewResolver에게 View의 물리 주소를 요청한다.
<br>

- 현재 여러 버전의 handler(controller)가 있다고 가정한다. 
- 회원가입 형식(new-form)을 처리하는 Handler는 version3(V3)로 수행되길 원한다. 
- 회원저장(save)과 회원조회(members)는 version4의 Handler로 수행되길 원한다.

> Commit: https://github.com/evelyn82ny/Model-View-Controller-pattern/commit/67c40963166056f468b2150eddae2e6b2d70130c

![png](/_img/210629_getHandler.png)

- Mapping value가 Object이므로 다양한 Interface를 받을 수 있다.
- 요청받은 uri를 수행하는 Handler를 handlerMappingMap에서 찾는다.
<br>

![png](/_img/210629_getAdapter.png)

- handlerAdapterList에 여러 HandlerAdapter 객체를 담는다. 
- getHandler method로 Handler를 찾았다면 현재 FrontController에서 원하는 반환 객체로 수행하기 위해 Handler를 처리할 수 있는 Handler adapter를 찾는다.

<br>

## interface HandlerAdapter

![png](/_img/210629_interfaceHandlerAdapter.png)

- ```supports()```로 전달받은 Handler가 해당 Adapter로 처리가능한지 판별한다.
- ```handle()```로 Front Controller가 ModelAndView 객체를 반환받기 원한다면 파라미터로 전달받은 Handler를 처리한 뒤 ModelAndView 객체로 반환한다. 
- HandlerAdapter를 **Interface**로 구현하여 모든 버전에서 동일한 요구사항을 처리하도록 확장성을 높인다.

<br>

## HandlerAdapter 상속하는 ControllerV4

ControllerV4가 View 만 반환하는 controller 이다.

![png](/_img/210629_controller_handle.png)

> Commit: https://github.com/evelyn82ny/Model-View-Controller-pattern/commit/ecbaac157658dd6b18b8a76fdcf5f1d95c87e1e0

- ```supports()``` 로 ```instanceof``` 연산자로 전달받은 Handler를 ControllerV4로 형변환 가능한지 판별한다. 
- ```handle()``` method가 실행되면 파라미터로 전달받은 Handler를 ControllerV4로 casting 한다. 
- ```createParamMap()``` 메소드로 request의 파라미터를 얻고 Model 객체를 생성한다.
<br>

- ControllerV4로 casting된 handler는 paramMap과 Model 객체로 역할을 수행한다.
- 현재 Front Controller에서 반환받기 원하는 객체는 ModelAndView 이므로 전달받은 View와 데이터 처리된 Model로 ```ModelAndView```를 생성하여 저장한 뒤 반환한다. 
- 이렇게 HandlerAdapter를 통해 어떤 Handler를 전달받아도 요구하는 객체로 반환하며 Adapter를 통해서만 Handler(controller)를 호출할 수 있다.

<br>

## FrontController 의 최종 Service code

```java
@Override
protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
    Object handler = getHandler(request);  // 요청된 Handler(Controller)를 얻는다.
    if(handler == null){
        response.setStatus(HttpServletResponse.SC_NOT_FOUND);
        return;
    }

    MyHandlerAdapter adapter = getHandlerAdapter(handler); // handler(controller)에 적용가능한 Adapter를 얻는다.

    ModelView mv = adapter.handle(request, response, handler); // Adapter를 통해 어떤 Handler든지 같은 객체로 반환받는다.

    String viewName = mv.getViewName(); // new-form : 논리 주소
    MyView view = viewResolver(viewName); // /WEB-INF/views/new-form : 물리 주소
    view.render(mv.getModel(), request, response);
}

private Object getHandler(HttpServletRequest request) {
        String requestURI = request.getRequestURI();
        return handlerMappingMap.get(requestURI);
        }

private MyHandlerAdapter getHandlerAdapter(Object handler) {
    for (MyHandlerAdapter adapter : handlerAdapterList) {
        if(adapter.supports(handler)){
            return adapter;
        }
    }
    throw new IllegalArgumentException("handler adapter를 찾을 수 없습니다." + handler);
}
private MyView viewResolver(String viewName) {
    return new MyView("/WEB-INF/views/" + viewName + ".jsp");
}
```

> Commit: https://github.com/evelyn82ny/Model-View-Controller-pattern/commit/ecbaac157658dd6b18b8a76fdcf5f1d95c87e1e0

1. ```getHandler()```로 요청된 uri를 처리할 수 있는 handler(controller)를 찾는다.
2. ```getHandlerAdapter()```로 Handler의 Adapter 역할을 해줄 수 있는 Adapter를 찾는다.
3. Adapter를 통해 역할을 수행한 뒤 요구하는 객체로 반환받는다.
4. ViewResolver로 view를 물리 주소로 변경한다.
5. View에서 rendering 한다.
<br>

# 정리

![png](/_img/mvc_history.png)

클라이언트 요청을 단순히 비즈니스 로직과 뷰 로직으로 한번에 처리하던 단순한 과정이 많은 과정을 통해 세분화되었다. **비즈니스 로직과 view 로직을 분리**했고 **Interface 와 구현도 분리**시켜 유지보수에 좋은 아키텍처가 되었다.

아래는 실제 MVC architecture 이며, 여러 과정을 통해 만들어진 아키텍처와 동일하다.

![png](/_img/mvc_architecture.png)

FrontController 는 실제 spring 에서 DispatcherServlet 이다.