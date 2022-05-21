# 스프링부트 예외 처리
BasicErrorController가 기본적으로 동작하면서 예외처리를 해준다.  
컨트롤러에서 예외가 발생했을 때 MediaType이 text/html인 경우 에러 넘버에 일치하는 html 뷰를 반환해주고 MediaType이 application/json인 경우 responseEntity로 Map을 반환한다.
