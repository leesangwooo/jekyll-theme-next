---
title: java servlet filter(2) 
description: 자바 서블릿 필터, 파일 업로드를 위한 마임설정
categories:
 - labs
tags: java
---


javax.servlet.Filter

## 웹어플리케이션의 파일 업로드 과정
1. 뷰 레이어에서 `<form>` 태그의 `<input type='file' name='fileUpload' />`사용
```
	<form method="post"
		action="${pageContext.request.contextPath }/file/upload" // 파일 업로드를 처리할 서블릿
		enctype="multipart/form-data"> // 인코딩 타입 설정 
	
		전송할 파일 : <input type="file" name="selFile" value="" />
		파일 업로더 : <input type="text" name="uploader" value="" />
		<input type="submit" value="전송" />
	
	</form>
```
	- *`enctype="multipart/form-data"`*
		- form의 input data를 여러개의 part로 구분하여 관리한다.
		- form태그의 기본적인 enctype인 `application/x-www-form-urlencoded`은 파라미터를 문자열로 처리하므로,
		바이너리 타입의 파일 전송을 가능하게 하기 위해선 multipart/form-data를 사용한다.

2. filter에서 request에 대한 wrapper 클래스를 사용하여 mime 설정에 따라 parameter 파싱 
	- mime 설정이 `multipart`인 경우 문자 데이터는 파싱되지 않는다.
	  따라서 문자열 엔트리로 구성된 parameterMap의 값이 채워지지 않으므로,
      wrapper 클래스를 생성하여 request객체에 파라미터를 다시 채워주는 작업이 필요하다.
      
	- 파라미터를 채워준 request wrapper 객체를 doFilter메소드의 매개변수에 request 객체 대신 전달한다.
	 : `chain.doFilter(wrapper, response);`


## 코드 샘플 1. request의 wrapper 클래스
```java
/**
 * multipart 요청을 파싱하고, 비어있는 parameterMap을 채우기 위한 Wrapper 객체.
 */
public class FileUploadRequestWrapper extends HttpServletRequestWrapper {
	// 문자데이터용
	private Map<String, String[]> parameterMap;
	
	// 파일 데이터용
	private Map<String, FileItem[]> fileItemMap;
	public FileUploadRequestWrapper(HttpServletRequest request) throws FileUploadException, UnsupportedEncodingException {
		this(request, -1, null);
	}

	public FileUploadRequestWrapper(HttpServletRequest request,
			int sizeThreshold, File repository) throws FileUploadException, UnsupportedEncodingException {
		super(request);
		parameterMap = new HashMap<String, String[]>();
		fileItemMap = new HashMap<String, FileItem[]>();
		
		// queryString 데이터를 확보하기 위한 코드
		parameterMap.putAll(request.getParameterMap());
		parseRequest(request, sizeThreshold, repository);
	}

	private void parseRequest(HttpServletRequest request, int sizeThreshold,
			File repository) throws FileUploadException, UnsupportedEncodingException {

		DiskFileItemFactory itemFactory = new DiskFileItemFactory();
		if(sizeThreshold!=-1){
			itemFactory.setSizeThreshold(sizeThreshold);
		}
		if(repository!=null){
			itemFactory.setRepository(repository);
		}
		
		ServletFileUpload handler = new ServletFileUpload(itemFactory);
		List<FileItem> itemList = handler.parseRequest(request);
		for(FileItem item : itemList){
			String fieldName = item.getFieldName();
			if(item.isFormField()){
				String value = item.getString(request.getCharacterEncoding());
				// 이미 같은 이름의 파라미터가 있는지여부를 확인
				String[] already = parameterMap.get(fieldName);
				String[] array = null;
				if(already==null){
					array = new String[]{value};
				} else {
					array = new String[already.length+1];
					System.arraycopy(already, 0, array, 0, already.length);
					array[array.length-1] = value;
				}
				parameterMap.put(fieldName, array);
			} else {
				
				FileItem[] already = fileItemMap.get(fieldName);
				FileItem[] array = null;
				if(already==null){
					array = new FileItem[]{item};
				} else {
					array = new FileItem[already.length+1];
					System.arraycopy(already, 0, array, 0, already.length);
					array[array.length-1] = item;
				}
				fileItemMap.put(fieldName, array);
				
			} // if~else end
		} // for end
		
	}
	//  필터링한 parameterMap을 이용하여 request의 파라미터 관련 메서드들을 오버라이드한다.
	@Override
	public Map<String, String[]> getParameterMap() {
		return parameterMap;
	}
	
    // 파라미터의 getter와 유사하게 fileItemMap을 사용할 수 있도록 getter들을 생성 
	public FileItem getFileItem(String name){
		FileItem[] array = fileItemMap.get(name);
		FileItem item = null;
		if(array!=null && array.length > 0){
			item  = array[0];
		}
		return item;
	}
	
}
```
## 코드 샘플 2. 블라인드 필터
```java
/**
 * 파일 업로드 요청인지 여부를 확인하고, 원본 요청을 wrapper로 바꿔서 다음 필터에게 전달.
 */
public class FileUploadCheckFilter implements Filter {
	
	Logger logger = LoggerFactory.getLogger(this.getClass());

	@Override
	public void init(FilterConfig filterConfig) throws ServletException {
	}

	@Override
	public void doFilter(ServletRequest request, ServletResponse response,
			FilterChain chain) throws IOException, ServletException {

		// 뷰레이어에서 form 태그에 설정한 enctype 속성은
        // request의 content-type 헤더로 전송된다. 
		HttpServletRequest req = (HttpServletRequest) request;
		String mime = req.getHeader("content-type");

		// mime 속성값이 multipart인지 여부를 확인
		// if(mime!=null && mime.startsWith("multipart")){
        // 위 코드는 apatch의 fileUpload api를 사용할 경우 아래와 같이 대체할 수 있다.
		if (ServletFileUpload.isMultipartContent(req)) {
        	
            // mime 설정이 multipart인 경우 문자열로 구성된 parameterMap의 값이 전달되지 않으므로
        	// wrapper 클래스를 생성하여 request객체 대신 전달한다.
			FileUploadRequestWrapper wrapper = null;
			try {
				wrapper = new FileUploadRequestWrapper(req);
				chain.doFilter(wrapper, response);
			} catch (FileUploadException | UnsupportedEncodingException e) {
				throw new CommonException(e);
			}
		} else {
        	// mime 설정이 multipart이 아닌 경우 그대로 필터를 통과
			chain.doFilter(request, response);
		}
	}

	@Override
	public void destroy() {
		logger.info("{} 필터 소멸", getClass().getSimpleName());
	}
}
```

## 필터를 통해 wrapping한 request 객체를 사용하기
```java
// wrapping 인스턴스 확인
if(req instanceof FileUploadRequestWrapper){
	// 다운 캐스팅하여 getter호출
    FileItem[] items  = ((FileUploadRequestWrapper) req).getFileItems("files");
    ...
```
