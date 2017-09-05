---
title: 用poi解析带分组级别的excel文件
date: 2017-09-01 21:14:26
tags:
    - poi
    - excel
    - java
---
# 使用poi解析带有分组级别的excel文件，并且递归封装为java父子级对象
最近从客户那里拿了一份excel数据，需要导入到数据库，心想挺简单的，所以忙了一天，到晚上才开始弄，结果发现excel带有组合信息，搞了好一会才弄好，所以总结一下
  首先excel带组合信息的话，仔细看会发现，子类们的父类都是他们的上一条，所以可以考虑先读出数据，再使用递归的方式封装
  下面附上代码：
```javascript
<!--  excel poi -->
		<dependency>
			<groupId>org.apache.poi</groupId>
			<artifactId>poi</artifactId>
			<version>3.5-FINAL</version>
		</dependency>
		<dependency>
			<groupId>org.apache.poi</groupId>
			<artifactId>poi-ooxml</artifactId>
			<version>3.5-FINAL</version>
		</dependency>
```


```javascript
package com.example.demo.FileUtils;

import com.example.demo.po.ExcelObject;
import org.apache.poi.hssf.usermodel.HSSFCell;
import org.apache.poi.hssf.usermodel.HSSFRow;
import org.apache.poi.hssf.usermodel.HSSFSheet;
import org.apache.poi.hssf.usermodel.HSSFWorkbook;
import org.apache.poi.ss.usermodel.Cell;
import org.apache.poi.xssf.usermodel.XSSFCell;
import org.apache.poi.xssf.usermodel.XSSFRow;
import org.apache.poi.xssf.usermodel.XSSFSheet;
import org.apache.poi.xssf.usermodel.XSSFWorkbook;

import java.io.*;
import java.util.ArrayList;
import java.util.List;

/**
 * excel 操作
 * Created by cuilb3 on 2017/8/29.
 */
public class Excel {

    public static void main(String[] args) {
        String path = "E:\\26305.xlsm";//源文件
        try {
            List<List<String>> result = new Excel().readXlsx(path);
            System.out.println(result.size());
            List<ExcelObject> excelObjects = new ArrayList<>();
            for (int i = 0; i < result.size(); i++) {
                ExcelObject excelObject = new ExcelObject();
                List<String> model = result.get(i);
                excelObject.setId(model.get(0));
                excelObject.setName(model.get(1));
                excelObject.setMark(model.get(2));
                excelObject.setType(model.get(5));
                excelObject.setLevel(model.get(model.size()-1));
                excelObjects.add(excelObject);
                System.out.println("num1:" + model.get(0) + "--> name:" + model.get(1)+"--->"+model.get(model.size()-1));
            }
            List<ExcelObject> list = getGroup(0,excelObjects);

            File tmpfile = new File("E:\\k2\\sql\\excel_template_device_bom_etl.sql");
            BufferedWriter writer = new BufferedWriter(new FileWriter(tmpfile));
            //设置父类id和id,并写入文件sql语句
            LogInfo("0",list,writer);
            writer.flush();
            writer.close();
            System.out.println(list);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    //开始插入数据库的第一条的id,数据库中最后一条加上1
    static int num = 0;

    /**
    * 生产sql文件
    **/
    public static int  LogInfo(String parentId,List<ExcelObject> list,BufferedWriter writer) throws IOException {
        for(int i =0 ;i<list.size();i++){
            list.get(i).setId(num+"");
            list.get(i).setParentId(parentId);
 	    writer.write("insert into device (id,device_no,name,type,remark,is_root) values ('"+num+"','a_b_"+num+"','"+list.get(i).getName()+"','"+list.get(i).getType()+"','"+list.get(i).getMark()+"',0);" + "\n");
            writer.write("insert into device_bom (id,device_id,parent_id) values ('"+num+"','"+num+"','"+Integer.parseInt(parentId) + "');\n");
            num++;
            if(list.get(i).getChildren()!=null&&list.get(i).getChildren().size()>0){
                LogInfo(list.get(i).getId(),list.get(i).getChildren(),writer);
                continue;
            }
        }
        return 1;
    }

    /**
     * 获取分组树节信息
     * @param list
     * @return
     */
    static int startPoint = 0 ;
    public static  List<ExcelObject> getGroup(int start,List<ExcelObject> list){
        List<ExcelObject> excelObjects = new ArrayList<>();
        for(int i = start ;i<list.size();i++){
            ExcelObject excelObject = list.get(i);
            if(excelObjects.size()==0){
                excelObjects.add(excelObject);
                System.out.println("startPoint:"+startPoint+"    ---->  i="+i);
                startPoint++;
            }else if (excelObject.getLevel().equals(excelObjects.get(excelObjects.size()-1).getLevel())){
                excelObjects.add(excelObject);
                System.out.println("startPoint:"+startPoint+"    -->  i="+i);
                startPoint++;
            }else{
                //等级大，继续递归
                if(Integer.valueOf(excelObject.getLevel())>Integer.valueOf(excelObjects.get(excelObjects.size()-1).getLevel())){
                    List chidren  = getGroup(startPoint,list);
                    i=startPoint-1;
                    List oldchildren = excelObjects.get(excelObjects.size()-1).getChildren();
                    if(oldchildren!=null && oldchildren.size()>0){
                        oldchildren.addAll(chidren);
                        excelObjects.get(excelObjects.size()-1).setChildren(oldchildren);
                    }else{
                        excelObjects.get(excelObjects.size()-1).setChildren(chidren);
                    }
                }else{
                    return excelObjects;
                }
            }
        }
        return excelObjects;
    }


    /**
     *
     * @Title: readXls
     * @Description: 处理xls文件
     *  从代码不难发现其处理逻辑：
     * 1.先用InputStream获取excel文件的io流
     * 2.然后穿件一个内存中的excel文件HSSFWorkbook类型对象，这个对象表示了整个excel文件。
     * 3.对这个excel文件的每页做循环处理
     * 4.对每页中每行做循环处理
     * 5.对每行中的每个单元格做处理，获取这个单元格的值
     * 6.把这行的结果添加到一个List数组中
     * 7.把每行的结果添加到最后的总结果中
     * 8.解析完以后就获取了一个List<List<String>>类型的对象了
     *
     * @param @param path
     * @param @return
     * @param @throws Exception    设定文件
     * @return List<List<String>>    返回类型
     * @throws
     *
     */
    private List<List<String>> readXls(String path) throws Exception {
        InputStream is = new FileInputStream(path);
        // HSSFWorkbook 标识整个excel
        HSSFWorkbook hssfWorkbook = new HSSFWorkbook(is);
        List<List<String>> result = new ArrayList<List<String>>();
        int size = hssfWorkbook.getNumberOfSheets();
        // 循环每一页，并处理当前循环页
        for (int numSheet = 0; numSheet < size; numSheet++) {
            // HSSFSheet 标识某一页
            HSSFSheet hssfSheet = hssfWorkbook.getSheetAt(numSheet);
            if (hssfSheet == null) {
                continue;
            }
            // 处理当前页，循环读取每一行
            for (int rowNum = 1; rowNum <= hssfSheet.getLastRowNum(); rowNum++) {
                // HSSFRow表示行
                HSSFRow hssfRow = hssfSheet.getRow(rowNum);
                int minColIx = hssfRow.getFirstCellNum();
                int maxColIx = hssfRow.getLastCellNum();
                List<String> rowList = new ArrayList<String>();
                // 遍历改行，获取处理每个cell元素
                for (int colIx = minColIx; colIx < maxColIx; colIx++) {
                    // HSSFCell 表示单元格
                    HSSFCell cell = hssfRow.getCell(colIx);
                    if (cell == null) {
                        continue;
                    }
                    rowList.add(getStringVal(cell));
                }
                result.add(rowList);
            }
        }
        return result;
    }

    /**
     *
     * @Title: readXlsx
     * @Description: 处理Xlsx文件
     * @param @param path
     * @param @return
     * @param @throws Exception    设定文件
     * @return List<List<String>>    返回类型
     * @throws
     */
    private List<List<String>> readXlsx(String path) throws Exception {
        InputStream is = new FileInputStream(path);
        XSSFWorkbook xssfWorkbook = new XSSFWorkbook(is);
        List<List<String>> result = new ArrayList<List<String>>();
        // 循环每一页，并处理当前循环页
        for (XSSFSheet xssfSheet : xssfWorkbook) {
            if (xssfSheet == null) {
                continue;
            }
            // 处理当前页，循环读取每一行
            for (int rowNum = 1; rowNum <= xssfSheet.getLastRowNum(); rowNum++) {
                XSSFRow xssfRow = xssfSheet.getRow(rowNum);
                int minColIx = xssfRow.getFirstCellNum();
                int maxColIx = xssfRow.getLastCellNum();
                List<String> rowList = new ArrayList<String>();
                for (int colIx = minColIx; colIx < maxColIx; colIx++) {
                    XSSFCell cell = xssfRow.getCell(colIx);
                    if (cell == null) {
                        continue;
                    }
                    rowList.add(cell.toString());
                }
                rowList.add(String.valueOf(xssfRow.getCTRow().getOutlineLevel()));
                result.add(rowList);
            }
        }
        return result;
    }


    // 存在的问题
    /*
     * 其实有时候我们希望得到的数据就是excel中的数据，可是最后发现结果不理想
     * 如果你的excel中的数据是数字，你会发现Java中对应的变成了科学计数法。
     * 所以在获取值的时候就要做一些特殊处理来保证得到自己想要的结果
     * 网上的做法是对于数值型的数据格式化，获取自己想要的结果。
     * 下面提供另外一种方法，在此之前，我们先看一下poi中对于toString()方法:
     *
     * 该方法是poi的方法，从源码中我们可以发现，该处理流程是：
     * 1.获取单元格的类型
     * 2.根据类型格式化数据并输出。这样就产生了很多不是我们想要的
     * 故对这个方法做一个改造。
     */
    /*public String toString(){
        switch(getCellType()){
            case CELL_TYPE_BLANK:
                return "";
            case CELL_TYPE_BOOLEAN:
                return getBooleanCellValue() ? "TRUE" : "FALSE";
            case CELL_TYPE_ERROR:
                return ErrorEval.getText(getErrorCellValue());
            case CELL_TYPE_FORMULA:
                return getCellFormula();
            case CELL_TYPE_NUMERIC:
                if(DateUtil.isCellDateFormatted(this)){
                    DateFormat sdf = new SimpleDateFormat("dd-MMM-yyyy")
                    return sdf.format(getDateCellValue());
                }
                return getNumericCellValue() + "";
            case CELL_TYPE_STRING:
                return getRichStringCellValue().toString();
            default :
                return "Unknown Cell Type:" + getCellType();
        }
    }*/

    /**
     * 改造poi默认的toString（）方法如下
     * @Title: getStringVal
     * @Description: 1.对于不熟悉的类型，或者为空则返回""控制串
     *               2.如果是数字，则修改单元格类型为String，然后返回String，这样就保证数字不被格式化了
     * @param @param cell
     * @param @return    设定文件
     * @return String    返回类型
     * @throws
     */
    public static String getStringVal(HSSFCell cell) {
        switch (cell.getCellType()) {
            case Cell.CELL_TYPE_BOOLEAN:
                return cell.getBooleanCellValue() ? "TRUE" : "FALSE";
            case Cell.CELL_TYPE_FORMULA:
                return cell.getCellFormula();
            case Cell.CELL_TYPE_NUMERIC:
                cell.setCellType(Cell.CELL_TYPE_STRING);
                return cell.getStringCellValue();
            case Cell.CELL_TYPE_STRING:
                return cell.getStringCellValue();
            default:
                return "";
        }
    }
}

```
