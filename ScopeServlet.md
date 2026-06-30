# ScopeServlet - 초심자부터 면접 대비까지 정리

## 1. 초심자용 실생활 비유
* **비유**: 개인 정보를 보관하고 공유하는 **서류 수첩들의 유효 범위**
* **설명**: 
  - 웹 서비스에서 데이터를 임시로 보관하는 수첩(Scope)은 보관 기간과 공유 범위에 따라 크게 3가지로 나뉩니다.
  1. **Request Scope (`request.setAttribute`)**: **1회용 포스트잇**
     - 특정 질문(요청)을 하고 답변(응답)을 받을 때까지만 살아있는 일회용 메모입니다. 포워드(forward)를 통해 다른 부서(JSP)에 메모를 고대로 넘겨줄 순 있지만, 최종 답변이 완료되면 쓰레기통에 버려집니다.
  2. **Session Scope (`session.setAttribute`)**: **개인 전용 다이어리**
     - 특정 고객(브라우저)이 로그인해서 로그아웃하거나 창을 닫을 때까지(세션 종료) 유지되는 개인 수첩입니다. 쿠키를 통해 철수, 영희를 구별하여 각각의 다이어리를 관리합니다.
  3. **Application Scope (`servletContext.setAttribute`)**: **사무실 공용 게시판**
     - 서버가 켜져서 꺼질 때까지 유지되며, 모든 고객과 모든 서블릿이 다 같이 보고 쓰는 거대한 공용 게시판입니다.

---

## 2. 중급자용 실제 일어나는 일과 의존성, 문법 특성

### Redirect와 Forward의 차이 및 내부 동작

#### 1. Redirect (`resp.sendRedirect("URL")`)
* **동작**: 서버가 클라이언트에게 `302 Found` 상태 코드와 함께 `Location` 헤더에 이동할 URL을 실어 보냅니다.
* **특성**: 브라우저는 이 응답을 받고 **새로운 HTTP 요청**을 완전히 다시 보냅니다. 브라우저의 주소창이 최종 URL로 변경되며, 완전히 새로운 `Request`/`Response` 객체가 생성됩니다. 따라서 이전 요청에 담겨있던 request 속성(`setAttribute`)은 유지되지 않고 사라집니다.

#### 2. Forward (`req.getRequestDispatcher("path").forward(req, resp)`)
* **동작**: 서버 내부에서 `RequestDispatcher`가 제어권을 지정한 리소스(JSP 등)로 이동시킵니다.
* **특성**: 이 모든 과정이 **서버 내부에서 은밀하게 일어나므로** 브라우저는 제어권 이동 여부를 알지 못해 주소창의 URL이 변경되지 않습니다. 기존의 `Request`와 `Response` 객체가 그대로 재사용되어 전달되므로, 서블릿 단계에서 `request.setAttribute()`를 통해 담은 객체 데이터를 JSP에서 성공적으로 꺼내서 렌더링할 수 있습니다.

### 서블릿의 3대 데이터 영역 (Scope)
1. **Request Scope (`HttpServletRequest`)**:
   * 하나의 클라이언트 요청 주기와 수명을 같이합니다. 포워딩 시 데이터를 넘겨주는 바인더 역할을 수행합니다.
2. **Session Scope (`HttpSession`)**:
   * 개별 웹 브라우저(클라이언트)별 고유 영역입니다. 브라우저가 첫 요청을 보낼 때 컨테이너가 발급한 세션 ID(`JSESSIONID`) 쿠키를 활용하여 구분합니다. 주로 로그인 사용자 정보, 장바구니 등을 저장합니다.
3. **Application Scope (`ServletContext`)**:
   * 웹 애플리케이션 당 1개만 생성되며, 애플리케이션의 시작부터 종료까지 생명주기가 유지됩니다. 전체 서블릿 및 모든 클라이언트가 데이터를 공유할 수 있는 전역 컨텍스트를 제공합니다.

---

## 3. 면접 준비를 위한 예상 질문

### Q1. Redirect와 Forward의 차이점에 대해 자세히 설명해주세요.
* **A.** 두 방식의 핵심 차이는 '클라이언트의 인지 여부'와 '요청 객체의 유지 여부'입니다. **Redirect**는 서버가 브라우저에게 재요청 명령(302 Found)을 내려 브라우저가 새로운 URL로 요청을 다시 보내게 하므로, 주소창이 바뀌고 기존의 Request/Response 객체는 완전히 소멸합니다. 반면 **Forward**는 서버 내부에서 기존의 Request/Response 객체를 유지한 채 다른 페이지로 제어권을 넘기므로 브라우저 주소창은 유지되고 서블릿에서 설정한 Request attribute 데이터 역시 그대로 JSP 등에서 활용할 수 있습니다.

### Q2. Servlet의 3대 Scope(Request, Session, Application)의 생명주기와 주 용도를 비교해주세요.
* **A.** 
  - **Request Scope**는 클라이언트 요청이 들어와 응답이 나갈 때까지만 유효하며, 서블릿에서 비즈니스 로직을 처리한 결과를 뷰(JSP)에 넘겨줄 때 사용합니다.
  - **Session Scope**는 개별 사용자(브라우저)가 접속해 종료할 때까지 유효하며, 로그인 정보나 사용자별 설정 등을 보관할 때 사용합니다.
  - **Application Scope**는 웹 애플리케이션이 톰캣 등에 업로드되어 가동될 때부터 종료될 때까지 유효하며, 모든 사용자와 서블릿이 공유해야 하는 전역적인 환경 설정값 등을 저장할 때 사용합니다.

### Q3. JSP에서 `request.getParameter()`와 `request.getAttribute()`의 차이는 무엇인가요?
* **A.** `getParameter()`는 클라이언트가 보낸 HTTP 요청 파라미터(URL의 Query String 또는 HTTP Body의 Form 데이터 등)를 읽어오기 위한 API로, 리턴 타입은 항상 String입니다. 반면 `getAttribute()`는 서버 내부(예: 서블릿)에서 다른 컴포넌트(예: JSP)로 데이터를 넘겨주기 위해 `request` 객체에 임의로 설정한 속성 값을 가져오기 위한 API로, 어떠한 자바 객체 타입(`Object`)이든 담고 반환받을 수 있습니다.
