# Spring MVC

- Model, View 그리고 Controller로 분리한 Design pattern

> 해당 포스트에 작성된 코드는 실제 정의된 코드가 아니라 개념을 익히기 위해 유사하게 구현했습니다.

<br>

회원 데이터를 받고 저장한 뒤 성공적으로 저장되었다고 노출시키는 view 를 servlet 이나 JSP 만으로 구현가능하다.

- Servlet으로 개발 시 자바 코드에 View를 위한 HTML를 추가
- JSP으로 View에 대한 HTML을 작성하고 동적 변경이 필요한 부분을 추가 구현

![png](/_img/210629_simpleRequest.png)

- 이 경우 비즈니스 로직과 view 렌더링이 1개의 파일 안에서 구현한다.
- 하지만 비즈니스 로직만 변경할 때는 view 코드를 변경할 필요가 없고 반대의 경우도 마찬가지다. 
- 즉, 개발 시 **라이프 사이클이 다른 경우엔 분리해서 작성해야 유지 보수가 좋아진다**. 
- 이를 위해 MVC 패턴이 만들어졌다.
- 
<br>

# controller 와 view 역할 구분

![png](/_img/210628_mvc1.png)

1. Client가 요청하면 **Controller**는 HTTP 요청을 받아 파라미터를 검증한다.
2. Controller는 **Business 로직**을 실행시켜 원하는 데이터를 처리한다.
3. Business 로직을 통해 처리된 데이터를 **View**에 전달하기 위해 **Model에 저장**한다. 이때 Model은 ```HttpServletRequest``` 객체이고 내부에 데이터 저장소를 갖고있다.
4. Controller는 **view 로직**을 호출한다.
5. view는 Model에 담겨있는 데이터를 사용해 노출할 화면을 HTML로 생성한다.
6. 클라이언트의 요청에 대한 응답을 한다.

![png](/_img/210628_mvcMemberSaveServletCode.png)

> Commit: https://github.com/evelyn82ny/Model-View-Controller-pattern/commit/adc971694cbfde64daa9cf05584bc48e0b0645fa

회원을 저장하는 과정을 MVC 패턴에 적용하면 위와 같은 코드가 된다.

- 요청받은 파라미터를 검증 및 파싱하고
- business 로직을 수행한 뒤
- Model에 데이터를 저장하고
- view 로직를 통해 화면을 노출한다.

<br>

## Model

- 데이터를 저장하고 조회하는 model 은 HttpServletRequest 객체를 사용한다. 
- HttpServletRequest 내부에 데이터 저장소가 있고
- ```setAttribute()```, ```getAttribute()``` 으로 데이터 보관 및 조회한다.

<br>

## WEB-INF

- MVC 패턴을 통해 controller 로 비즈니스 로직을 처리한 다음 view가 호출되는 것을 원한다.
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

# FrontController (DispatcherServlet)

MVC 패턴을 적용한 결과 **Controller의 역할과 View를 렌더링 하는 역할을 명확하게 구분**했다.<br>
회원을 저장, 조회하는 등 많은 기능을 위 코드와 같이 작성할 수 있지만 다음과 같은 문제가 발생한다.<br>

- view path 가 모두 비슷한 경로일 때나 view 의 경로가 바뀌거나 다른 뷰로 변경한다면 모든 코드를 다 변경해야 한다.
- ```RequestDispatcher```에서 forward도 모든 코드에서 사용되니 중복처리 하면 간단해질 수 있다.
- ```HttpServletRequest```, ```HttpServletResponse``` 의 모든 기능을 사용하지 않으며 test case 작성도 어렵다.
- 즉, 기능이 복잡해질수록 공통으로 처리해야하는 부분이 많아지기 때문에 컨트롤러 호출 전에 공통 기능을 처리하면 문제를 해결할 수 있다.

이를 해결하기 위ㅎ Front Controller(프론트 컨트롤러) 패턴을 사용한다.
<br>

![png](/_img/front_controller.png)

- FrontController Servlet 1개로 모든 클라이언트 요청을 받는다. 
- FrontController는 요청에 맞는 Controller를 호출한다. 
- FrontController는 공통 처리하는 servlet이므로 controller에선 더이상 servlet을 사용하지 않아도 된다. 
- 즉, controller에서 ```@WebServlet``` 으로 url 을 mapping 하거나 ```HttpServlet```을 상속받지 않아도 된다.

실제 spring MVC 의 **DispatcherServlet**이 front controller 패턴으로 구현되어 있다.

<br>

![png](/_img/front_controller_mapping.png)

- HTTP request에 맞는 controller를 찾기 위해 front controller는 mapping정보에서 controller를 찾는다. 
- Front Controller는 url과 mapping되는 controller를 호출하고 jsp forward하여 html 응답한다.

다형성을 이용하기 위해 controller를 Interface로 구현하고 회원 저장, 조회 등 각각의 기능은 controller interface 를 상속받아 구현했다. front controller servlet 코드는 다음과 같다.

![png](/_img/210628_frontControllerServlet_code.png)

> Commit: https://github.com/evelyn82ny/Model-View-Controller-pattern/commit/a66abb5b9977a1ed2fd867f2302580ba9a27a4ea

## 모든 요청 Front Controller로 받기

- ```@WebServlet``` 의 urlPattern 을 ```/front-controller/*``` 로 작성하면 ```/front-controller``` 하위 url은 해당 servlet으로 들어온다. 
- 모든 요청을 FrontController로 받는 과정이 수행된다.

<br>

## HttpServletRequest getRequestURI()

- controllerMap 에 **{ key: mapping URI , value: 호출될 controller }** 로 여러 controller를 저장했다. 
- ```getRequestURI()```로 요청된 URI를 읽고, controllerMap에서 mapping된 controller를 찾는다.
- controller 를 찾았다면 해당 controller 의 process 를 수행한다.
- 만약 controller 를 찾지 못했다면 404 상태 코드(SC_NOT_FOUND)를 반환한다.

<br>

# View 분리

위 과정을 통해 요청에 맞는 controller를 찾고 공통 부분 처리를 위한 Front Controller servlet를 추가했다.<br>
하지만 controller에서 view를 직접 Forward하는 상태이므로 controller와 view가 완전히 분리된 상태가 아니다.

![png](/_img/architecture_that_call_render_method.png)

- controller 에서 해당되는 view를 Front controller로 반환한다. 
- 해당 controller 에서 반환된 view 를 호출하는 방식이며 controller 와 view 가 완전히 분리된다.
- 여기서 말하는 view 는 실제 spring MVC 의 View 이다.

> Commit: > Commit: https://github.com/evelyn82ny/Model-View-Controller-pattern/commit/0961c4eaeeea88a42bfbcb984206dbe6484a3c11

![png](/_img/210628_view_code.png)

Front Controller에 의해 View가 호출되면 request에서 ```getRequestURI()```로 uri 를 읽어 forward 로직을 수행하여 JSP 가 실행된다.

![png](/_img/210628_view_frontControllerServlet.png)

- 모든 controller 는 해당되는 View 객체를 생성하여 FrontControllerServlet 에 반환한다. 
- FrontControllerServlet 은 받은 View 의 ```render()```을 실행한다. 
- front controller 를 통해 View 객체의 ```render()``` 호출을 일관되게 처리하게 된다.

<br>

# httpServlet 대신 Model 로 변경 및 ViewResolver로 물리주소 받기

## 중복되는 view 경로 제거

- 현재 모든 controller 에서 view 경로를  ```return new View("/WEB-INF/views/save-result.jsp");```인 물리 경로로 반환한다. 
- **/WEB-INF/views** 는 중복되는 부분이먀 만약 수많은 view 코드가 있는 views 폴더를 다른 폴더로 옮긴다면 모든 controller 코드를 변경해야한다.

이를 해결하기 위해 controller는 해당되는 view의 논리 이름만 반환하고 front controller에서 받은 view 논리 이름을 물리 주소로 변경한다.

- view 물리 주소 : /WEB-INF/views/save-result.jsp
- view 논리 이름 : save-result

<br>

## servlet 종속성 제거

- 여러 controller는 파라미터로 ```httpServletRequest```, ```httpServletResponse``` 를 받도록 했지만 항상 사용하지 않았다. 
- 또한, controller는 servlet 기술이 아닌 처리 해야하는 데이터만 필요하므로 servlet 자체를 넘겨주는게 불필요해 보인다.
- 이를 해결하기 위해 ```HttpServlet``` 객체를 Model로 사용하지 않고 Model객체를 생성하여 이용한다. 
- 즉, controller가 servlet 기술을 알 필요없이 데이터만 처리할 수 있도록 servlet 종속성을 제거한다.

![png](/_img/architecture_with_view_resolver_applied.png)

> Commit: https://github.com/evelyn82ny/Model-View-Controller-pattern/commit/a4afa239426fdcb0c2ca0e7ac3c4778daf2afb2e

![png](/_img/210629_modelAndView_code.png)

- controller 는 FrontController 에게 {View 논리 이름, 데이터가 담긴 Model}인 **ModelAndView** 객체를 반환한다.
- ModelAndView 코드를 보면 String으로 View의 논리 이름을, Map<String, Object>로 Model를 만들어 데이터를 담는다.

![png](/_img/210629_modelAndView_controllerCode.png)

- 위 코드는 회원을 저장하는 Controller이며 전달받는 파라미터가 httpServlet 이 아니라 요청을 처리하기 위해 필요한 데이터이다. 
- 받은 데이터로 controller 의 역할을 수행한 뒤 반환하기 위한 ModelAndView 객체를 생성한다.
- ModelAndView 객체의 View 에는 controller 에 해당되는 **view 논리이름**을 저장하고 Model에는 데이터 이름과 데이터를 저장한다.
- controller에 해당되는 view 논리 이름과 처리한 데이터가 저장된 ModelAndView 객체를 FrontController 로 반환한다.
- view 논리 이름만 반환하면 되므로 view 의 경로가 바껴도 Controller를 수정할 필요가 없다.

![png](/_img/210629_viewResolver_code.png)

위 코드는 ModelAndView 와 ViewResolver 을 적용했다.

### createParamMap

- FrontController Servlet 은 클라이언트의 요청에 따른 HTTP request message 를 처리하여 ```HttpServletRequest```, ```HttpServletResponse``` 객체를 전달받는다. 
- 전달받은 httpServlet을 그대로 controller에게 전달하지 않고 처리해야하는 데이터만 뽑아 전달한다.
- createParamMap 메소드로 ```HttpServletRequest```에서 데이터만 뽑아 Map<String, String>에 저장한 뒤 controller에게 전달한다. 
- controller는 전달받은 데이터를 처리해 ModelAndView 객체로 반환한다.

### viewResolver

- ViewResolver 는 controller 에게 받은 view 논리 이름을 물리 주소 변경한다. 
- 이처럼 view 경로가 변경면 controller 코드를 수정할 필요없이 viewResolver 메소드만 변경하면 된다. 
- view 에서 렌더링 하기 위해 Model 도 같이 넘겨 준다.

![png](/_img/210629_viewRender.png)

> Commit: https://github.com/evelyn82ny/Model-View-Controller-pattern/commit/a4afa239426fdcb0c2ca0e7ac3c4778daf2afb2e

- frontController Servlet 에 있는 Model 을 파라미터로 받는다. 
- ```HttpServletRequest```와 ```HttpServletResponse``` 객체를 forward 하기 위해 ```setAttribute()```로 Model data를 ```HttpServletRequest``` 객체에 저장한다.

<br>


# ModelAndView 대신 return View 로 편리성 증가

- Model 객체를 통해 Servlet 종속성을 제거했고 ViewResolver 로 view 경로 중복을 제거하여 자신의 역할에만 충실할 수 있는 설계가 되었다. 
- 하지만 Controller 에서 ModelAndView 객체를 생성하고 저장 후 반환해야하는 번거로움을 존재한다. 
- 이전보다 많이 편리해졌지만 더 편리해지기 위해 Model 객체를 FrontController에서 만들고 Controller에게 Model 객체를 파라미터로 전달한다.

![png](/_img/architecture_in_which_the_view_is_returned_from_controller.png)

- FrontController Servlet 에서 Map<String, Object> model 객체를 만든다. 
- 실행해야 할 controller 에게 model 객체를 파라미터로 넘겨주고 Controller에게 View 논리이름을 반환받는다.

> Commit: https://github.com/evelyn82ny/Model-View-Controller-pattern/commit/7ac12856bdb8fdd44459efb907278382133ddba3

![png](/_img/210629_returnModel_code.png)

- 렌더링을 위해 view render 메소드 호출 시 frontController Servlet에 있는 model을 그대로 넘겨준다.
- Controller는 파라미터로 전달받은 Model에 데이터를 저장한다. 
- 번거롭게 ModelAndView 객체를 생성할 필요없이 View 논리 이름만 반환하면 된다.

![png](/_img/210629_controller_parameterModel_code.png)

- FrontController Servlet 에서 model 을 파라미터로 넘기고, View 의 논리 이름만 반환하는 방식으로 변경했다.
- 이는 Controller를 구현하는 개발자가 다른 부분을 신경쓰지 않고 controller의 역할에만 집중하여 코드작성할 수 있게 된다.

<br>

# 다양한 interface 를 처리하는 HandlerAdapter

어떤 부분은 ModelAndView 객체를 반환하는 Interface 기반의 controller를 사용하고 싶고, 다른 부분은 View 만 반환하는 interface 기반의 controller 를 사용하고 싶을 때 Adapter 개념을 적용한다.

- controller 명칭을 handler 로 변경한다. 
- Adapter 로 어떤 형태든 상관없이 모두 처리가능하기 때문에 controller 보다 더 넓은 범위인 handler 로 변경한다.

![png](/_img/architecture_with_handler_adapter_applied.png)

1. HTTP 요청에 맞는 handler (controller)를 찾는다.
2. handler 를 처리할 수 있는 handler adapter 를 찾는다.
3. 반환받고자하는 객체를 handler adapter 에게 요구한다.
4. handler adapter 는 handler 를 호출해 요청사항을 수행한다.
5. handler 로 수행된 결과를 front controller 가 요구하는 객체로 생성하여 반환한다.
6. ViewResolver 에게 view 의 물리 주소를 요청한다.

- 현재 여러 버전의 handler(controller) 가 있다고 가정한다. 
- 회원가입 형식(new-form) 을 처리하는 handler 는 version3(V3)로 수행되길 원한다. 
- 회원저장(save)과 회원조회(members)는 version4의 handler로 수행되길 원한다.

> Commit: https://github.com/evelyn82ny/Model-View-Controller-pattern/commit/67c40963166056f468b2150eddae2e6b2d70130c

![png](/_img/210629_getHandler.png)

- mapping value 가 Object 이므로 다양한 interface 를 받을 수 있다.
- 요청받은 uri 를 수행하는 handler 를 handlerMappingMap 에서 찾는다.

![png](/_img/210629_getAdapter.png)

- handlerAdapterList 에 여러 HandlerAdapter 객체를 담는다. 
- getHandler method로 handler를 찾았다면 현재 FrontController 에서 원하는 반환 객체로 수행하기 위해 handler를 처리할 수 있는 handler adapter를 찾는다.

<br>

## interface HandlerAdapter

![png](/_img/210629_interfaceHandlerAdapter.png)

- supports 로 전달받은 handler 가 해당 adapter 로 처리가능한지 판별한다.
- handle method 로 front controller 가 ModelAndView 객체를 반환받기 원한다면 파라미터로 전달받은 handler 를 처리한 뒤 ModelAndView 객체로 반환한다. 
- handlerAdapter 를 **interface**로 구현하여 모든 버전에서 동일한 요구사항을 처리하도록 확장성을 높인다.

<br>

## HandlerAdapter 상속하는 ControllerV4

ControllerV4가 View 만 반환하는 controller 이다.

![png](/_img/210629_controller_handle.png)

> Commit: https://github.com/evelyn82ny/Model-View-Controller-pattern/commit/ecbaac157658dd6b18b8a76fdcf5f1d95c87e1e0

- supports 로 instanceof 연산자로 전달받은 handler를 ControllerV4로 형변환 가능한지 판별한다. 
- handle method가 실행되면 파라미터로 전달받은 handler를 ControllerV4로 casting 한다. 
- createParamMap 메소드로 request 의 파라미터를 얻고 Model 객체를 생성한다.
<br>

- ControllerV4로 casting 된 handler 는 paramMap 과 Model 객체로 역할을 수행한다.
- 현재 Front Controller 에서 반환받기 원하는 객체는 ModelAndView 이므로 전달받은 View 와 데이터 처리된 Model 로 ModelAndView 를 생성하여 저장한 뒤 반환한다. 
- 이렇게 HandlerAdapter 를 통해 어떤 handler 를 전달받아도 요구하는 객체로 반환하며 Adapter 를 통해서만 Handler(controller)를 호출할 수 있다.

<br>

## FrontController 의 최종 service code

![png](/_img/210629_adapter_code.png)

> Commit: https://github.com/evelyn82ny/Model-View-Controller-pattern/commit/ecbaac157658dd6b18b8a76fdcf5f1d95c87e1e0

1. getHandler 메소드로 요청된 uri 를 처리할 수 있는 handler(controller)를 찾는다.
2. getHandlerAdapter 메소드로 handler 의 adapter 역할을 해줄 수 있는 Adapter 를 찾는다.
3. adapter 를 통해 역할을 수행한 뒤 요구하는 객체로 반환받는다.
4. ViewResolver 로 view 를 물리 주소로 변경한다.
5. View 에서 렌더링한다.
<br>

# 정리

![png](/_img/mvc_history.png)

클라이언트 요청을 단순히 비즈니스 로직과 뷰 로직으로 한번에 처리하던 단순한 과정이 많은 과정을 통해 세분화되었다. **비즈니스 로직과 view 로직을 분리**했고 **Interface 와 구현도 분리**시켜 유지보수에 좋은 아키텍처가 되었다.

아래는 실제 MVC architecture 이며, 여러 과정을 통해 만들어진 아키텍처와 동일하다.

![png](/_img/mvc_architecture.png)

FrontController 는 실제 spring 에서 DispatcherServlet 이다.