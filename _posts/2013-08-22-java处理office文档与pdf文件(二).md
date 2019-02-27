该部分主要内容：文件上传，以及office文件和pdf的html处理，以及提取text
```
// 根据服务器的文件保存地址和原文件名创建目录文件全路径
		File file = this.getFile();
		String url = "";
		String tempFile = "";
		String fileFolder = "";	//上传文件路径
		String hz = "";
		String oldOrgFileId = null;
		Long oldId = knowledge.getZsk_zskID();
		if(null != oldId && 0 != oldId){
			oldOrgFileId = knowledge.getOrgFileId();
		}
		
		if(null != file){
			// 截取扩展名
			hz = fileFileName.substring(fileFileName.lastIndexOf("."),fileFileName.length());
			String zskCode = knowledge.getZsk_Code();
			fileFolder = createNewFile(this.savePath,zskCode);
			// 上传的文件在服务器中的全路径
			url = fileFolder + "\\" + fileFileName;
			//1、文件上传
			FileUtils.copyFile(file, new File(url));	
			
			//2、文件转化为html
			tempFile = createNewFile(this.tempPath,zskCode);
			String htmlStr = "";
			if(hz.equals(".pdf")){
				htmlStr = "<html><body>" +
				"<embed src='"+fileFileName+"' width='100%' height='100%'></embed>" +
				"</body></html>";
			}else{
				String dstHtml = tempFile+"\\"+zskCode+".html";
				//删除文件夹下所有文件及子文件夹
				FileUtil.deleteChildFile(new File(tempFile));
				
				changeDocToHtml(hz, url, dstHtml);
				htmlStr = FileUtil.htmlToStr(dstHtml);
			}
			knowledge.setContentHtml(htmlStr);
			Clob htmlColb=Hibernate.createClob(htmlStr);
			knowledge.setZsk_Description(htmlColb);
			
			//3、获取上传文件对应的文本内容
			String docContent = findDocContent(hz, url);
			knowledge.setContentText(docContent);
			Clob docContentClob=Hibernate.createClob(docContent);
			knowledge.setZsk_Text(docContentClob);
			
			String orgFileId = new GUID().toString();	//知识库原文件对应的标识
			knowledge.setOrgFileId(orgFileId);
			knowledge.setZsk_ContentType(1);
		}else{
			Clob htmlColb = Hibernate.createClob(htmlArea);
			Clob textClob = Hibernate.createClob(htmlArea.replaceAll("</?[^>]+>", ""));
			knowledge.setZsk_Description(htmlColb);
			knowledge.setContentHtml(htmlArea);
			knowledge.setZsk_Text(textClob);
			knowledge.setContentText(htmlArea);
			knowledge.setZsk_ContentType(2);
		}
		
		//添加时处理
		if(null == oldId || 0 ==  oldId){
			//to--do  需要在后期重新处理 当前用户
			if(null == knowledge.getZsk_Author() || "".equals(knowledge.getZsk_Author())){	//当前用户
				knowledge.setZsk_Author(SessionUtil.getTSysAgent().getCagentname());
			}
			knowledge.setZsk_RegisterTime(new Date());
		}
		//to---do 
		knowledge.setZsk_LastMender(1L);
		knowledge.setZsk_ModifyTime(new Date());
		
		KnowLedgeOtherContion ko = new KnowLedgeOtherContion();
		ko.setFileContentType(fileContentType);
		ko.setFileFileName(fileFileName);
		ko.setOldId(oldId);
		ko.setTempFile(tempFile);
		ko.setUrl(url);
		ko.setOldOrgFileId(oldOrgFileId);
		
		knowUploadServiceImp.saveOrUpdateKnowledge(knowledge,ko);
```
将office转化为html
```
/**
	 * 将word,excel,ppt,pdf转化为html
	 * @param hz
	 * @param url
	 * @param dstHtml
	 */
	private void changeDocToHtml(String hz, String url, String dstHtml) {
		if("pdf".equalsIgnoreCase(hz)){
			
		}else if(".xls".equalsIgnoreCase(hz) || ".xlsx".equalsIgnoreCase(hz)){
			DocToHtml.getInstance().ExceltoHtml(url,dstHtml);
		}else if(".doc".equalsIgnoreCase(hz) || ".docx".equalsIgnoreCase(hz)){
			DocToHtml.getInstance().WordtoHtml(url,dstHtml);
		}else if(".ppt".equalsIgnoreCase(hz) || ".pptx".equalsIgnoreCase(hz)){
			DocToHtml.getInstance().PPTtoHtml(url, dstHtml);
		}
	}
```
将word,wxcel,ppt另存为html的方法
```
public boolean WordtoHtml(String srcFile, String dstFile) {
		ComThread.InitSTA();
		ActiveXComponent activexcomponent = new ActiveXComponent("Word.Application");
		String s2 = srcFile;
		String s3 = dstFile;
		boolean flag = false;
		try {
			activexcomponent.setProperty("Visible", new Variant(false));
			Dispatch dispatch = activexcomponent.getProperty("Documents").toDispatch();
			Dispatch dispatch1 = Dispatch.invoke(dispatch, "Open", 1,
					new Object[] { s2, new Variant(false), new Variant(true) },
					new int[1]).toDispatch();
			Dispatch.invoke(dispatch1, "SaveAs", 1, new Object[] { s3,new Variant(8) }, new int[1]);
			Variant variant = new Variant(false);
			Dispatch.call(dispatch1, "Close", variant);
			flag = true;
		} catch (Exception exception) {
			log.error("word转化为html出错-->"+exception.getMessage());
		} finally {
			activexcomponent.invoke("Quit", new Variant[0]);
			ComThread.Release();
			ComThread.quitMainSTA();
		}
		return flag;
	}

	public boolean PPTtoHtml(String srcFile, String dstFile) {
		ComThread.InitSTA();
		ActiveXComponent activexcomponent = new ActiveXComponent( "PowerPoint.Application");
		boolean flag = false;
		try {
			Dispatch dispatch = activexcomponent.getProperty("Presentations")
					.toDispatch();
			Dispatch dispatch1 = Dispatch.call(dispatch, "Open", srcFile,
					new Variant(-1), new Variant(-1), new Variant(0))
					.toDispatch();
			Dispatch.call(dispatch1, "SaveAs", dstFile, new Variant(12));
//			Variant variant = new Variant(-1);
			Dispatch.call(dispatch1, "Close");
			flag = true;
		} catch (Exception exception) {
			log.error("ppt转化为html出错-->"+exception.getMessage());
		} finally {
			activexcomponent.invoke("Quit", new Variant[0]);
			ComThread.Release();
			ComThread.quitMainSTA();
		}
		return flag;
	}

	public boolean ExceltoHtml(String s, String s1) {
		 ComThread.InitSTA();
		 ActiveXComponent activexcomponent = new
		 ActiveXComponent("Excel.Application");
		 boolean flag = false;
		 try
		 {
			 activexcomponent.setProperty("Visible", new Variant(false));
			 Dispatch dispatch = activexcomponent.getProperty("Workbooks").toDispatch();
			 Dispatch dispatch1 = Dispatch.invoke(dispatch, "Open", 1, new Object[] {
					 s, new Variant(false), new Variant(true)}, new int[1]).toDispatch();
			 Dispatch.call(dispatch1, "SaveAs", s1, new Variant(44));
			 Variant variant = new Variant(false);
			 Dispatch.call(dispatch1, "Close", variant);
			 flag = true;
		 }catch(Exception exception){
			 log.error("excel转化为html出错-->"+exception.getMessage());
		 }finally{
			 activexcomponent.invoke("Quit", new Variant[0]);
			 ComThread.Release();
			 ComThread.quitMainSTA();
		 }
		 return flag;
	}
```
获取office文件以及pdf的文本内容
```
private String findDocContent(String hz, String url) {
		String docContent = null;
		File file = new File(url);
		if(".pdf".equalsIgnoreCase(hz)){
			docContent = GetDocText.getDocTextInta().getTextFromPdf(file);
		}else if(".xls".equalsIgnoreCase(hz) || ".xlsx".equalsIgnoreCase(hz)){
			docContent = GetDocText.getDocTextInta().getTextFromExcel(file);
		}else if(".doc".equalsIgnoreCase(hz) || ".docx".equalsIgnoreCase(hz)){
			docContent = GetDocText.getDocTextInta().getTextFromWord(file);
		}else if(".ppt".equalsIgnoreCase(hz) || ".pptx".equalsIgnoreCase(hz)){
			docContent = GetDocText.getDocTextInta().getTextFromPPT(file);
		}
		return docContent;
	}
```
具体的实现方法
```
/**
	 * 从word文件获取文本内容
	 * 
	 * @param wordFile
	 * @return word文件的文本内容
	 */
	public String getTextFromWord(File wordFile) {
		String wordText = "";
		InputStream is = null;
		try {  
            //word 2003： 图片不会被读取  
            is = new FileInputStream(wordFile);
            String fileName = wordFile.getName();
		    String hz = fileName.substring(fileName.lastIndexOf("."),fileName.length());
		    if(".doc".equals(hz)){
		    	WordExtractor ex = new WordExtractor(is);  
		    	wordText = ex.getText();
		    }else{
		    	OPCPackage opcPackage = POIXMLDocument.openPackage(wordFile.getAbsolutePath());
		    	POIXMLTextExtractor extractor = new XWPFWordExtractor(opcPackage);  
		    	wordText = extractor.getText();
		    }
              
        } catch (Exception e) {  
            e.printStackTrace();  
        }finally{
        	if(is != null){
        		try {
					is.close();
				} catch (IOException e) {
					e.printStackTrace();
				}
        	}
        }
		return wordText;
	}

	/**
	 * 从excel获取文本内容
	 * 
	 * @param excelFile
	 * @return Excel文件的文本内容
	 */
	public String getTextFromExcel(File excelFile) {
		String text = "";
		InputStream in = null;
		try {
			//创建相关的文件流对象
			in = new FileInputStream(excelFile);
		    //声明相关的工作薄对象
			Workbook wb =null;
		    //声明相关的excel抽取对象
		    ExcelExtractor extractor=null;
		    String fileName = excelFile.getName();
		    String hz = fileName.substring(fileName.lastIndexOf("."),fileName.length());
		    
		    if(hz.equals(".xls"))//针对2003版本
		    {
		    	//创建excel2003的文件文本抽取对象
		    	wb=new HSSFWorkbook(new POIFSFileSystem(in));
		    	extractor =new org.apache.poi.hssf.extractor.ExcelExtractor((HSSFWorkbook)wb);
		    }else{ //针对2007版本
		    	wb = new  XSSFWorkbook(in);
		    	//创建excel2007的文件文本抽取对象
		    	extractor =new XSSFExcelExtractor((XSSFWorkbook)wb);
		    }
		    
		    extractor.setFormulasNotResults(false);
		    //是否抽象sheet页的名称
		    extractor.setIncludeSheetNames(true);
		    //是否抽取cell的注释内容
		    extractor.setIncludeCellComments(true);
		    //获取相关的抽取文本信息
		    text = extractor.getText();
		} catch (FileNotFoundException e) {
			e.printStackTrace();
		} catch (IOException e) {
			e.printStackTrace();
		}finally{
			if(in != null){
				try {
					in.close();
				} catch (IOException e) {
					e.printStackTrace();
				}
			}
		}
		
		return text;
	}
	/**
	 * 从ppt获取文本内容
	 * 
	 * @param pptFile
	 * @return ppt文件的文本内容
	 */
	public String getTextFromPPT(File pptFile){
		String pptText = null;
		FileInputStream fin = null;
		try {
			fin = new FileInputStream(pptFile);
			String fileName = pptFile.getName();
			String hz = fileName.substring(fileName.lastIndexOf("."),fileName.length());
			if(".ppt".equals(hz)){
				QuickButCruddyTextExtractor qct = new QuickButCruddyTextExtractor(fin);
				pptText = qct.getTextAsString();
			}else{
				OPCPackage opcPackage = POIXMLDocument.openPackage(pptFile.getAbsolutePath());
				XSLFPowerPointExtractor pptExtractor = new XSLFPowerPointExtractor(opcPackage);
				pptText = pptExtractor.getText();
			}
		} catch (FileNotFoundException e) {
			e.printStackTrace();
		} catch (IOException e) {
			e.printStackTrace();
		} catch (XmlException e) {
			e.printStackTrace();
		} catch (OpenXML4JException e) {
			e.printStackTrace();
		}finally{
			if(null != fin){
				try {
					fin.close();
				} catch (IOException e) {
					e.printStackTrace();
				}
			}
		}
		return pptText;
	}
	/**
	 * 从pdf文件获取文本内容
	 * 
	 * @param pdfFile
	 * @return pdf文件的文本内容
	 */
	public String getTextFromPdf(File pdfFile){
		String result = null;
		FileInputStream is = null;
		PDDocument document = null;
		try{
			is = new FileInputStream(pdfFile);
			PDFParser parser = new PDFParser(is);
			parser.parse();
			document = parser.getPDDocument();
			PDFTextStripper stripper = new PDFTextStripper();
			result = stripper.getText(document);
		}catch(FileNotFoundException e){
			e.printStackTrace();
		}catch(IOException e){
			e.printStackTrace();
		}finally{
			if(is != null){
				try{
					is.close();
				}catch(IOException e){
					e.printStackTrace();
				}
			}
			if(document != null){
				try{
					document.close();
				}catch(IOException ex){
					ex.printStackTrace();
				}
			}
		}
		return result;
	}
	/**
	 * 
	 * @param txtFile
	 * @return  返回txt的内容
	 */
	public String getTextFromTxt(File txtFile){
		FileReader fr;
		StringBuffer buff = new StringBuffer();
		try {
			fr = new FileReader(txtFile);
			BufferedReader br = new BufferedReader(fr);
			String temp = null;
			while((temp = br.readLine()) != null){
				buff.append(temp + "\r\n"); 
			}
			br.close();
		} catch (FileNotFoundException e) {
			e.printStackTrace();
		} catch (IOException e) {
			e.printStackTrace();
		}
		
		return buff.toString();
	}
```
对带有clob字段的实体save时，直接调用hibernate的save即可。ojdbc14.jar。
更新时的处理  如下：
```
public void updateKnowledge(Knowledge knowledge) {
		 try {
		    knowledge.setZsk_Description(Hibernate.createClob(" "));
		    knowledge.setZsk_Text(Hibernate.createClob(" "));
		    update(knowledge);
		    flush();
		    
		    getSession().refresh(knowledge, LockMode.UPGRADE);
		    
		    SerializableClob htmlSc=(SerializableClob)knowledge.getZsk_Description();
		    SerializableClob textSc=(SerializableClob)knowledge.getZsk_Text();
		    Clob htmlWrapclob=htmlSc.getWrappedClob();
		    Clob textWrapclob=textSc.getWrappedClob();
		    CLOB htmlClob2=(CLOB)htmlWrapclob;
		    CLOB textClob2=(CLOB)textWrapclob;
		    Writer htmlWriter=htmlClob2.getCharacterOutputStream();
		    htmlWriter.write(knowledge.getContentHtml());
		    htmlWriter.close();
		    
		    Writer textWriter=textClob2.getCharacterOutputStream();
		    textWriter.write(knowledge.getContentText());
		    textWriter.close();
		    
		    update(knowledge);
		  } catch (RuntimeException re) {
		    throw re;
		  } catch (SQLException e) {
		    e.printStackTrace();
		  } catch (IOException e) {
		    e.printStackTrace();
		  }
	}
```
上面几步做完，基本可以完成上传以及存入数据库，以及对带有clob文件的更新。
需要的环境  windows，jacob-1.17-M2-x64 具体的jacob下载和配置 参照网络。poi-3.9