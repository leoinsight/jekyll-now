---
layout: post
title: Jasper Reports 사용법
---

Spring Boot와 연동한 Jasper Reports 설정 및 사용법을 기록.

## 환경설정 (Spring Boot 1.5.1, Jasper Studio 6.4.0final 기준)

### pom.xml
```xml
<!--...-->
	<repositories>
		<!-- local -->
		<repository>
			<id>local-repo</id>
			<name>local Repository</name>
			<url>file://${project.basedir}/local-repo</url>
		</repository>
	</repositories>

	<dependencies>
	<!--...-->
		<dependency>
		    <groupId>net.sf.jasperreports</groupId>
		    <artifactId>jasperreports</artifactId>
		    <version>6.4.0</version>
		</dependency>
		<dependency>
		    <groupId>org.apache.xmlgraphics</groupId>
		    <artifactId>xmlgraphics-commons</artifactId>
		    <version>2.2</version>
		</dependency>
		<!-- local repo -->
		<dependency>
			<groupId>net.sf.jasperreports.font.extension</groupId>
			<artifactId>NANUMGOTHIC</artifactId>
			<version>1.0</version>
		</dependency>
		<dependency>
			<groupId>net.sf.jasperreports</groupId>
			<artifactId>jasperreports-functions</artifactId>
			<version>6.4.0</version>
		</dependency>
	</dependencies>
<!--...-->
```  
<br/>

*[참고] local repository*  
![local-repo]({{ site.baseurl }}/images/local-repo.png)


### WebConfiguration.java (@Configuration 클래스)
```java
//...
	@Bean
	public JasperReportsViewResolver getJasperReportsViewResolver() {
		JasperReportsViewResolver resolver = new JasperReportsViewResolver();
	    resolver.setPrefix("classpath:/jasperreports/");
	    resolver.setSuffix(".jrxml");
	    
	    resolver.setViewNames("RPT_*");
	    resolver.setReportDataKey("datasource");
	    resolver.setSubReportDataKeys("subdata");
	    resolver.setViewClass(JasperReportsMultiFormatView.class);
	    return resolver;
	}
//...
```

### ReportController.java
```java
//...
	@GetMapping("/{reportName}/{inspSeqNum}")
	public ModelAndView reportByFormat(@PathVariable("reportName") String reportName,
			@PathVariable("inspSeqNum") Long inspSeqNum, @RequestParam("format") String format, ModelAndView mv) throws Exception {
	
		// List (Sample Code)
		List<imgVO> imgList = service.getImgList(...);
		List<Map<String, String>> imgPathList = new ArrayList<Map<String, String>>();
		for (int i = 0; i < imgList.size(); i++) {
			Map<String, Object> map = new HashMap<String, Object>();
			map.put("attPath", imgList.get(i).getAttPath());
			imgPathList.add(map);
		}
		
		mv.setViewName(reportName);		// jrxml 파일명
		mv.addObject("format", format);	// "pdf"
		mv.addObject("datasource", new JREmptyDataSource());
		mv.addObject("imgPathList", new JRBeanCollectionDataSource(imgPathList));
		
		return mv;
	}
//...
```
<br/>

*[참고] 호출 URL 예시*
> localhost:8080/rest/reports/RPT_Book1/999?format=pdf


## Jasper Studio 설정

### 프로젝트 하위 폴더생성 및 build path 설정
1. src/main/resources/jasperreports 생성  
![jasper-studio-project]({{ site.baseurl }}/images/jasper-studio-project.png)

2. 프로젝트 Properties > Java Build Path > Source > Add Folder... > resources 폴더 선택, Allow output folders for source folders    
![jasper-studio-buildpath]({{ site.baseurl }}/images/jasper-studio-buildpath.png)

### pdf 한글 관련 설정
Jaspersoft Studio > Preferences > Fonts > Add
```
Family Name: NANUMGOTHIC
TrueType(.ttf): ${project.basedir}\src\main\resources\static\font\NANUMGOTHIC.TTF
PDF Encoding: Identity-H (Unicode with horizontal writing)
Embed this font in PDF document (check)
Export to Local Repository (${project.basedir}\local-repo) 이하 groupId\artifactId
	ex) D:\project\bsa\workspace\boot-web\local-repo\net\sf\jasperreports\font\extension\NANUMGOTHIC\1.0\NANUMGOTHIC-1.0.jar
```

### Report 구성
1. 단일 Report (List) 구성 방법 (RPT_TEST_LIST1.jrxml)

	0) ReportController에서 넘어오는 Model Object의 attributeName 이 *imgPathList* 라고 가정
	- imgPathList 의 type은 ```List<Map<>> 또는 List<VO객체>```

	1) Parameters > Create Parameter
	- Name: imgPathList
	- Class: net.sf.jasperreports.engine.JRBeanCollectionDataSource
	
	2) Create Dataset
	- Name: imgPathList
	- 생성된 Dataset(imgPathList)에 Field 추가 (attPath)
	
	3) Detail band 에 List Element 추가 (Palette > Basic Elements 에서 드래그앤드랍)
	- List Properties > Dataset Run 선택: imgPathList (생성한 Dataset)
	- Use a JRDataSource expression 설정: $P{imgPathList} (생성한 Parameter)
	
	4) List Contents 구성 (List 더블클릭)
	- List에 Image Element 추가 (Palette > Basic Elements 에서 드래그앤드랍)
	- Image Properties > Expression 값 설정: $F{attPath}

### Report Book 구성
1. Report Book 구성 방법 (RPT_TEST_Book1.jrxml)

	1) File > New > Jasper Report > Report Book 템플릿 선택

	2) Content > Add new book part > Select an existing report, Select a report file... > Workspace resource, Browse... > RPT_TEST_LIST1.jrxml 선택

	3) Connection > Use a JRDataSource expression, new net.sf.jasperreports.engine.JREmptyDataSource() 입력

	4) RPT_TEST_Book1의 파라미터 생성 (from Controller)
	- Parameters > Create Parameter
	- Name: imgPathList
	- Class: net.sf.jasperreports.engine.JRBeanCollectionDataSource
	
	5) RPT_TEST_LIST1에 넘길 파라미터 설정 (from Report Book)
	- Content RPT_TEST_LIST1.jasper 선택 > Properties > Data 탭 > Edit Parameters 버튼 클릭
	- Add 버튼 클릭 > Parameter Name: imgPathList (Report Book 파라미터), Parameter Expression: $P{evtImgPathList} 선택

