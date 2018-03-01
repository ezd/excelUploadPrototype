UI:

<!DOCTYPE html>
<html xmlns:th="http://www.thyemleaf.org">
<head>
</head>
<body>
<p th:text="${message}"></p>
	<form method="post" enctype="multipart/form-data"
	  action="/uploadExcelFile">
	    <input type="file" name="file" accept=".xls,.xlsx" /> <input
	      type="submit" value="Upload file" />
	</form>
	
	<div th:if="${candidates!=null}">
    <table id="myTableId" dt:table="true">
        <thead>
            <tr>
                <th>Vendor</th>
                <th>Candidate Name</th>
                <th>Skills</th>
                <th>BRM</th>
                <th>SPOC</th>
                <th>Date</th>
                <th>Location</th>
                <th>Special Skills</th>
            </tr>
        </thead>
        <tbody>
            <tr th:each="candidate : ${candidates}">
                <td th:text="${candidate?.vendor}">Kebede</td>
                <td th:text="${candidate?.candidate_name}">Kebede</td>
                <td th:text="${candidate?.Skills}">Kebede</td>
                <td th:text="${candidate?.BRM}">Kebede</td>
                <td th:text="${candidate?.SPOC}">Kebede</td>
                <td th:text="${candidate?.date}">Kebede</td>
                <td th:text="${candidate?.location}">Kebede</td>
                <td th:text="${candidate?.specialSkills}">Kebede</td>
            </tr>
        </tbody>
    </table>
</div>

</body>
</html>

Controller:
package com.example.controller;

import java.io.File;
import java.io.FileOutputStream;
import java.io.IOException;
import java.io.InputStream;
import java.util.ArrayList;
import java.util.Iterator;
import java.util.List;
import java.util.Map;

import javax.annotation.Resource;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.multipart.MultipartFile;

//@RestController
@Controller
public class ExcelController {

	private String fileLocation;
	
//	@Resource(name = "excelPOIHelper")
	@Autowired
	private ExcelPOIHelper excelPOIHelper;
	 
	@PostMapping("/uploadExcelFile")
	public String uploadFile(Model model, MultipartFile file) throws IOException {
	    if(file.getOriginalFilename().endsWith("xls")||file.getOriginalFilename().endsWith("xlsx")){
	    	InputStream in = file.getInputStream();
		    File currDir = new File(".");
		    String path = currDir.getAbsolutePath();
		    fileLocation = path.substring(0, path.length() - 1) + file.getOriginalFilename();
		    FileOutputStream f = new FileOutputStream(fileLocation);
		    int ch = 0;
		    while ((ch = in.read()) != -1) {
		        f.write(ch);
		    }
		    f.flush();
		    f.close();
		    model.addAttribute("message", "File: " + file.getOriginalFilename() 
		      + " has been uploaded successfully!");
	    }else{
	    	 model.addAttribute("message", "Incorrect file");
	    }
		
	    return "excel";
	}
	
	
	 
	@RequestMapping(value = "/readPOI")
	public String readPOI(Model model) throws IOException {
	
	 CandidateDto candidate=null;
	 boolean isNumber=false;
	 boolean isEmail=false;
	 boolean isDate=false;
	 List<String> rejectedData=new ArrayList<String>();
	 List<CandidateDto> candidates=new ArrayList<CandidateDto>();
	 String BRMEmail="";
	 String SPOCEmail="";
	 String degitRegex = "[0-9]*\\.?[0-9]*";
	 String emailRegex = "^[_A-Za-z0-9-\\+]+(\\.[_A-Za-z0-9-]+)*@[A-Za-z0-9-]+(\\.[A-Za-z0-9]+)*(\\.[A-Za-z]{2,})$";
	 String dateRegex = "(0?[1-9]|[12][0-9]|3[01])/(0?[1-9]|1[012])/((19|20)\\d\\d)";
	  if (fileLocation != null) {
	      if (fileLocation.endsWith(".xlsx") || fileLocation.endsWith(".xls")) {
	          Map<Integer, List<MyCell>> data
	            = excelPOIHelper.readExcel(fileLocation);
	          Iterator<Map.Entry<Integer,List<MyCell>>> it=data.entrySet().iterator();
	          it.next();
	          while(it.hasNext()){
	        	  Map.Entry<Integer,List<MyCell>> row=it.next();
	        		  if(!row.getValue().isEmpty()){
	        			  candidate=new CandidateDto();
	        			  candidate.setVendor(row.getValue().size()>=1?row.getValue().get(0).getContent():"");
	        			  candidate.setCandidate_name(row.getValue().size()>=2?row.getValue().get(1).getContent():"");
	        			  candidate.setSkills(row.getValue().size()>=3?row.getValue().get(2).getContent():"");
	        			  if(row.getValue().size()>=4){
	        				  isEmail=row.getValue().get(3).getContent().matches(emailRegex);
	        				  if(isEmail){
	        					  BRMEmail=row.getValue().get(3).getContent();
	        					  candidate.setBRM(BRMEmail);
//	        					  get user by email >> User user=
	        				  }else{
	        					  rejectedData.add("Vendor:"+candidate.getVendor()+" candidate:"+candidate.getCandidate_name()+" is rejected. Invalid BRM Email");
	        					  continue;
	        				  }
	        				  
	        			  }
	        			  if(row.getValue().size()>=5){
	        				  isEmail=row.getValue().get(4).getContent().matches(emailRegex);
	        				  if(isEmail){
	        					  SPOCEmail=row.getValue().get(4).getContent();
	        					  candidate.setSPOC(SPOCEmail);
//	        					  get user by email >> User user=
	        				  }else{
	        					  rejectedData.add("Vendor:"+candidate.getVendor()+" candidate:"+candidate.getCandidate_name()+" is rejected. Invalid SPCO Email");
	        					  continue;
	        				  }
	        				  
	        			  }
	        			  if(row.getValue().size()>=6){
	        				  isDate=row.getValue().get(5).getContent().matches(dateRegex);
	        				  if(isDate){
	        					  candidate.setDate(row.getValue().get(5).getContent());
//	        					  get user by email >> User user=
	        				  }else{
	        					  rejectedData.add("Vendor:"+candidate.getVendor()+" candidate:"+candidate.getCandidate_name()+" is rejected. Invalid Date");
	        					  continue;
	        				  }
	        				  
	        			  }
	        			  candidate.setLocation(row.getValue().size()>=7?row.getValue().get(6).getContent():"");
	        			  candidate.setSpecialSkills(row.getValue().size()>=8?row.getValue().get(7).getContent():"");
	        			  
	        			 
	        			  
	        			  candidates.add(candidate);
	        		  }
	          }
	          model.addAttribute("candidates", candidates);
	          System.out.println(rejectedData.size()+" data are rejected");
	          System.out.println(candidates.size()+" data accepted");
	          model.addAttribute("data", data);
	      } else {
	          model.addAttribute("message", "Not a valid excel file!");
	      }
	  } else {
	      model.addAttribute("message", "File missing! Please upload an excel file.");
	  }
	  return "excel";
	}
	
	@GetMapping("/uploadExcelFile")
	public String uploadPage(Model model){
		List<CandidateDto> candidates=null;
		model.addAttribute("persons",candidates);
		model.addAttribute("message", "start");
		return "excel";
	}
}

Helper class:

package com.example.controller;

import org.apache.poi.ss.usermodel.Cell;
import org.apache.poi.xssf.usermodel.XSSFCell;
import org.apache.poi.xssf.usermodel.XSSFCellStyle;
import org.apache.poi.xssf.usermodel.XSSFColor;
import org.apache.poi.xssf.usermodel.XSSFFont;
import org.apache.poi.xssf.usermodel.XSSFRow;
import org.apache.poi.xssf.usermodel.XSSFSheet;
import org.apache.poi.xssf.usermodel.XSSFWorkbook;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.stereotype.Component;
import org.apache.poi.hssf.usermodel.HSSFCell;
import org.apache.poi.hssf.usermodel.HSSFCellStyle;
import org.apache.poi.hssf.usermodel.HSSFFont;
import org.apache.poi.hssf.usermodel.HSSFRow;
import org.apache.poi.hssf.usermodel.HSSFSheet;
import org.apache.poi.hssf.usermodel.HSSFWorkbook;
import org.apache.poi.hssf.util.HSSFColor;
import org.apache.poi.ss.usermodel.DateUtil;

import java.io.File;
import java.io.FileInputStream;
import java.io.IOException;
import java.util.Map;
import java.util.HashMap;
import java.util.ArrayList;
import java.util.List;
import java.util.stream.IntStream;

@Component
public class ExcelPOIHelper {

    public Map<Integer, List<MyCell>> readExcel(String fileLocation) throws IOException {

        Map<Integer, List<MyCell>> data = new HashMap<>();
        FileInputStream fis = new FileInputStream(new File(fileLocation));

        if (fileLocation.endsWith(".xls")) {
            data = readHSSFWorkbook(fis);
        } else if (fileLocation.endsWith(".xlsx")) {
            data = readXSSFWorkbook(fis);
        }

        int maxNrCols = data.values().stream()
          .mapToInt(List::size)
          .max()
          .orElse(0);

        data.values().stream()
          .filter(ls -> ls.size() < maxNrCols)
          .forEach(ls -> {
              IntStream.range(ls.size(), maxNrCols)
                .forEach(i -> ls.add(new MyCell("")));
          });

        return data;
    }

    private String readCellContent(Cell cell) {
        String content;
        switch (cell.getCellTypeEnum()) {
        case STRING:
            content = cell.getStringCellValue();
            break;
        case NUMERIC:
            if (DateUtil.isCellDateFormatted(cell)) {
                content = cell.getDateCellValue() + "";
            } else {
                content = cell.getNumericCellValue() + "";
            }
            break;
        case BOOLEAN:
            content = cell.getBooleanCellValue() + "";
            break;
        case FORMULA:
            content = cell.getCellFormula() + "";
            break;
        default:
            content = "";
        }
        return content;
    }

    private Map<Integer, List<MyCell>> readHSSFWorkbook(FileInputStream fis) throws IOException {
        Map<Integer, List<MyCell>> data = new HashMap<>();
        HSSFWorkbook workbook = null;
        try {
            workbook = new HSSFWorkbook(fis);

            HSSFSheet sheet = workbook.getSheetAt(0);
            for (int i = sheet.getFirstRowNum(); i <= sheet.getLastRowNum(); i++) {
                HSSFRow row = sheet.getRow(i);
                data.put(i, new ArrayList<>());
                if (row != null) {
                    for (int j = 0; j < row.getLastCellNum(); j++) {
                        HSSFCell cell = row.getCell(j);
                        if (cell != null) {
                            HSSFCellStyle cellStyle = cell.getCellStyle();

                            MyCell myCell = new MyCell();

                            HSSFColor bgColor = cellStyle.getFillForegroundColorColor();
                            if (bgColor != null) {
                                short[] rgbColor = bgColor.getTriplet();
                                myCell.setBgColor("rgb(" + rgbColor[0] + "," + rgbColor[1] + "," + rgbColor[2] + ")");
                            }
                            HSSFFont font = cell.getCellStyle()
                                .getFont(workbook);
                            myCell.setTextSize(font.getFontHeightInPoints() + "");
                            if (font.getBold()) {
                                myCell.setTextWeight("bold");
                            }
                            HSSFColor textColor = font.getHSSFColor(workbook);
                            if (textColor != null) {
                                short[] rgbColor = textColor.getTriplet();
                                myCell.setTextColor("rgb(" + rgbColor[0] + "," + rgbColor[1] + "," + rgbColor[2] + ")");
                            }
                            myCell.setContent(readCellContent(cell));
                            data.get(i)
                                .add(myCell);
                        } else {
                            data.get(i)
                                .add(new MyCell(""));
                        }
                    }
                }
            }
        } finally {
            if (workbook != null) {
                workbook.close();
            }
        }
        return data;
    }

    private Map<Integer, List<MyCell>> readXSSFWorkbook(FileInputStream fis) throws IOException {
        XSSFWorkbook workbook = null;
        Map<Integer, List<MyCell>> data = new HashMap<>();
        try {

            workbook = new XSSFWorkbook(fis);
            XSSFSheet sheet = workbook.getSheetAt(0);

            for (int i = sheet.getFirstRowNum(); i <= sheet.getLastRowNum(); i++) {
                XSSFRow row = sheet.getRow(i);
                data.put(i, new ArrayList<>());
                if (row != null) {
                    for (int j = 0; j < row.getLastCellNum(); j++) {
                        XSSFCell cell = row.getCell(j);
                        if (cell != null) {
                            XSSFCellStyle cellStyle = cell.getCellStyle();

                            MyCell myCell = new MyCell();
                            XSSFColor bgColor = cellStyle.getFillForegroundColorColor();
                            if (bgColor != null) {
                                byte[] rgbColor = bgColor.getRGB();
                                myCell.setBgColor("rgb(" + (rgbColor[0] < 0 ? (rgbColor[0] + 0xff) : rgbColor[0]) + "," + (rgbColor[1] < 0 ? (rgbColor[1] + 0xff) : rgbColor[1]) + "," + (rgbColor[2] < 0 ? (rgbColor[2] + 0xff) : rgbColor[2]) + ")");
                            }
                            XSSFFont font = cellStyle.getFont();
                            myCell.setTextSize(font.getFontHeightInPoints() + "");
                            if (font.getBold()) {
                                myCell.setTextWeight("bold");
                            }
                            XSSFColor textColor = font.getXSSFColor();
                            if (textColor != null) {
                                byte[] rgbColor = textColor.getRGB();
                                myCell.setTextColor("rgb(" + (rgbColor[0] < 0 ? (rgbColor[0] + 0xff) : rgbColor[0]) + "," + (rgbColor[1] < 0 ? (rgbColor[1] + 0xff) : rgbColor[1]) + "," + (rgbColor[2] < 0 ? (rgbColor[2] + 0xff) : rgbColor[2]) + ")");
                            }
                            myCell.setContent(readCellContent(cell));
                            data.get(i)
                                .add(myCell);
                        } else {
                            data.get(i)
                                .add(new MyCell(""));
                        }
                    }
                }
            }
        } finally {
            if (workbook != null) {
                workbook.close();
            }
        }
        return data;
    }

}

Cell class:

package com.example.controller;

public class MyCell {
	private String content;
    private String textColor;
    private String bgColor;
    private String textSize;
    private String textWeight;
    
    public MyCell() {
    }

    public MyCell(String content) {
        this.content = content;
    }
    
    
	public String getContent() {
		return content;
	}
	public void setContent(String content) {
		this.content = content;
	}
	public String getTextColor() {
		return textColor;
	}
	public void setTextColor(String textColor) {
		this.textColor = textColor;
	}
	public String getBgColor() {
		return bgColor;
	}
	public void setBgColor(String bgColor) {
		this.bgColor = bgColor;
	}
	public String getTextSize() {
		return textSize;
	}
	public void setTextSize(String textSize) {
		this.textSize = textSize;
	}
	public String getTextWeight() {
		return textWeight;
	}
	public void setTextWeight(String textWeight) {
		this.textWeight = textWeight;
	}

}

Candidate profile DTO:
package com.example.controller;

import java.util.Date;

public class CandidateDto {
	
	private String vendor;
	private String candidate_name;
	private String Skills;
	private String BRM;
	private String SPOC;
	private String date;
	private String location;
	private String specialSkills;
	public String getVendor() {
		return vendor;
	}
	public void setVendor(String vendor) {
		this.vendor = vendor;
	}
	public String getCandidate_name() {
		return candidate_name;
	}
	public void setCandidate_name(String candidate_name) {
		this.candidate_name = candidate_name;
	}
	public String getSkills() {
		return Skills;
	}
	public void setSkills(String skills) {
		Skills = skills;
	}
	public String getBRM() {
		return BRM;
	}
	public void setBRM(String bRM) {
		BRM = bRM;
	}
	public String getSPOC() {
		return SPOC;
	}
	public void setSPOC(String sPOC) {
		SPOC = sPOC;
	}
	
	public String getDate() {
		return date;
	}
	public void setDate(String date) {
		this.date = date;
	}
	public String getLocation() {
		return location;
	}
	public void setLocation(String location) {
		this.location = location;
	}
	public String getSpecialSkills() {
		return specialSkills;
	}
	public void setSpecialSkills(String specialSkills) {
		this.specialSkills = specialSkills;
	}
}


pom file:
<properties>
		<poi.version>3.16-beta1</poi.version>
	</properties>
<dependency>
            <groupId>org.apache.poi</groupId>
            <artifactId>poi-ooxml</artifactId>
            <version>${poi.version}</version>
        </dependency>
        



