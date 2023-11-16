# [11/15] JSP&Servlet (DispatcherServlet 실습)

## Github Repository

- [project 보러가기](https://github.com/kyukong/SeSAC-Servlet-Framework)

## DispatcherServlet properties 분리

- 기존 실습 코드의 DispatcherServlet Controller 관리 역할 분리
- 기존 : DispatcherServlet 에서 Controller Mapping

    ```java
    @Override
     protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        String requestURI = request.getRequestURI();
        String contextPath = request.getContextPath();
        String action = requestURI.substring(contextPath.length());
        
        // 1. DispatcherServlet 에서 Controller Mapping
        AbstractController controller = null;
        ModelAndView mav = null;
        
        if (action.equals("/pilot/form")) {
      	  controller = new FormController();
      	  mav = controller.handleRequestInternal(request, response);
        } else if (action.equals("/pilot/process")) {
      	  controller = new ProcessController();
      	  mav = controller.handleRequestInternal(request, response);
        }
    		...
    }
    ```

- 변경 : properties 파일에서 Controller Mapping

    ```java
    public class DispatcherServlet extends HttpServlet {
       
       private Map<String, AbstractController> controllerMap = new HashMap<>();
       
       @Override
       public void init() throws ServletException {
          
          Properties prop = new Properties();
          
          try {
        	  prop.load(new FileInputStream(this.getClass().getResource("dispatcher-servlet.properties").getPath()));
             for(Object oKey : prop.keySet()) {
                String key = ((String)oKey).trim();
                Class<?> className = null;
                try {
                   className = Class.forName(prop.getProperty(key).trim());
                   controllerMap.put(key, (AbstractController) className.getConstructor().newInstance());
                   System.out.println("🧡 loaded : " + className + " 🧡");
                } catch (Exception e) {
                   e.printStackTrace();
                   System.out.println("💔 error : " + className + " 💔");
                }
             }
          } catch (Exception e) {
             e.printStackTrace();
          }
       }
    
    	@Override
    	protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
    	    String requestURI = request.getRequestURI();
    	    String contextPath = request.getContextPath();
    	    String action = requestURI.substring(contextPath.length());
    	    
    	    // 2. properties 파일에서 Controller Mapping
    	    try {
    	  	  AbstractController controller = controllerMap.get(action);
    		  if (controller == null) {
    			  throw new RuntimeException("RequestDispatcher에서 길을 잃었다네~");
    		  }
    		  ModelAndView mav = controller.handleRequestInternal(request, response);
    		  if (mav == null) {
    			  throw new RuntimeException("RequestDispatcher에서 길을 잃었다네~");
    		  }
    		     
    	     String viewName = mav.getViewName();
    	     if (viewName.startsWith("redirect:")) {
    	        response.sendRedirect(viewName.substring(9));
    	     } else {
    	        Map<String, Object> model = mav.getModel();
    	        for(String key : model.keySet()) {
    	           request.setAttribute(key, model.get(key));
    	        }            
    	        RequestDispatcher dispatcher = request.getRequestDispatcher(viewName);
    	        dispatcher.forward(request, response);
    	     }
    		} catch (IOException e) {
    			e.printStackTrace();
    		}
    	 }
    }
    ```

## CRUD 실습

- `/article/insert` : 게시글 입력 폼 출력
    - article.controller.ArticleInsert.java
- `/article/insertAction` : 게시글 생성, 입력 폼에서 전송된 파라미터를 테이블에 저장
    - article.controller.ArticleInsertAction.java
- `/article/list` : 게시글 리스트
    - article.controller.ArticleList.java
- `/article/article?no=10` : 게시글 상세보기
    - article.controller.ArticleDetail.java
- `/article/update?no=10` : 게시글 수정 폼 출력 (입력된 게시물 내용 출력)
    - article.controller.ArticleUpdate.java
- `/article/updateAction?no=10` : 게시글 수정, 수정 폼에서 전송된 파라미터를 테이블에 수정
    - article.controller.ArticleUpdateAction.java
- `/article/delete?no=10` : 게시글 삭제 폼 출력 (입력된 게시물 내용 출력)
    - article.controller.ArticleUpdate.java
- `/article/delete?no=10` : 게시글 삭제, 삭제 폼에서 전송된 파라미터로 게시글 삭제
    - article.controller.ArticleUpdateAction.java
