# FirstServlet - 초심자부터 면접 대비까지 정리

## 1. 초심자용 실생활 비유
* **비유**: 편지를 받아 처리하는 **우체국 창구 직원**
* **설명**: 
  - **웹 브라우저(클라이언트)**는 우체국을 방문해 편지를 보내는 **고객**입니다.
  - `@WebServlet("/first")`는 이 직원이 근무하는 **창구 번호(예: 1번 창구)**입니다. 고객이 1번 창구로 편지를 보내면, 1번 창구 전담 직원(`FirstServlet`)이 활성화되어 편지를 받습니다.
  - **`HttpServletRequest`**는 고객이 작성해서 보낸 **편지 봉투와 그 안의 내용물** (보낸 사람 정보, 요청 매개변수 등)입니다.
  - **`HttpServletResponse`**는 직원이 고객에게 보낼 **답장 편지지와 봉투**입니다. 직원은 여기에 인코딩 형식을 정하고 내용을 채워서(`resp.getWriter().println()`) 고객에게 돌려보냅니다.

---

## 2. 중급자용 실제 일어나는 일과 의존성, 문법 특성

### 서블릿 컨테이너(Tomcat)의 동작 과정
1. **요청 수신**: 클라이언트가 `/first` 경로로 HTTP GET 요청을 보내면, 서블릿 컨테이너(Tomcat)는 HTTP 요청 패킷을 파싱하여 `HttpServletRequest`와 `HttpServletResponse` 객체를 생성합니다.
2. **서블릿 매핑 및 생성**: 컨테이너는 URL 매핑 정보를 바탕으로 해당 요청을 처리할 서블릿인 `FirstServlet` 인스턴스가 메모리에 존재하는지 확인합니다. 싱글톤 형태로 관리되므로 존재하지 않는다면 최초 1회 인스턴스를 생성(로드 및 초기화)하고, 이미 존재한다면 기존 인스턴스를 재사용합니다.
3. **스레드 할당 및 실행**: 요청마다 독립적인 스레드를 할당하고, `service()` 메소드를 실행합니다. `service()`는 HTTP 요청 메소드(GET)를 판별하여 내부적으로 개발자가 재정의한 `doGet()` 메소드를 호출(디스패치)합니다.

### 의존성 (Dependencies)
* **Jakarta EE 9+**: 기존의 `javax.servlet` 패키지 스페이스가 Eclipse 재단으로 이관되면서 `jakarta.servlet` 패키지 스페이스로 변경되었습니다.
  - `jakarta.servlet.http.HttpServlet`, `jakarta.servlet.http.HttpServletRequest`, `jakarta.servlet.http.HttpServletResponse` 등을 상속 및 참조합니다.

### 문법 및 API 특성
* **`@WebServlet("/first")`**: XML 설정 파일(`web.xml`)을 수정하지 않고 어노테이션 정의만으로 서블릿을 컨테이너에 매핑시킵니다.
* **`resp.setContentType("text/html; charset=utf-8")`**: HTTP 응답 헤더의 `Content-Type`과 문자 인코딩을 지정하여 브라우저가 수신한 문자열 데이터를 깨짐 없이 해석하도록 합니다.
* **Java Text Blocks (`""" ... """`)**: HTML 형식의 동적 콘텐츠 데이터를 자바 내에서 멀티라인 템플릿 형태로 깔끔하게 작성하는 데 유용합니다.

---

## 3. 면접 준비를 위한 예상 질문

### Q1. Servlet이란 무엇이고 어떻게 작동하나요?
* **A.** 서블릿은 클라이언트의 요청을 처리하고, 이에 따른 동적인 콘텐츠(HTML, JSON 등)를 응답으로 돌려주는 자바 기반의 웹 컴포넌트입니다. 서블릿 컨테이너(Tomcat 등)가 서블릿의 전체 생명주기를 관리하며, 클라이언트의 요청이 들어오면 컨테이너가 스레드를 할당하고 `service()` 메소드를 거쳐 적절한 HTTP 메소드 처리 메소드(`doGet`, `doPost` 등)로 분기하여 처리합니다.

### Q2. `@WebServlet` 어노테이션의 역할은 무엇이며 기존 방식과의 차이점은 무엇인가요?
* **A.** `@WebServlet` 어노테이션은 서블릿 클래스를 서블릿 컨테이너에 등록하고 특정 URL 패턴과 매핑하는 역할을 합니다. 서블릿 3.0 이전에는 `web.xml` 설정 파일에 서블릿 이름과 매핑 패턴을 각각 명시적으로 등록해야 했으나, 어노테이션 도입 이후 코드 내부에서 직관적이고 빠르게 매핑 관리를 할 수 있게 되었습니다.

### Q3. `HttpServletRequest`와 `HttpServletResponse` 객체는 언제, 그리고 누가 생성하나요?
* **A.** 서블릿 컨테이너(WAS)가 클라이언트로부터 HTTP 요청을 수신할 때마다 해당 요청 정보를 담은 `HttpServletRequest` 객체와 클라이언트에 전달할 응답용 `HttpServletResponse` 객체를 동적으로 생성합니다. 이후 요청 처리를 담당하는 서블릿 인스턴스의 `service()` 메소드를 호출할 때 매개변수로 전달됩니다.
