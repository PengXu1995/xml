# xml
xmlStudentManageSystem

考试xml：
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<exam>
	<student examid="222" idcast="111">
		<name>张三</name>
		<location>北京</location>
		<grade>89</grade>
	</student>

	<student examid="444" idcast="333">
		<name>李四</name>
		<location>上海</location>
		<grade>91</grade>
	</student>
</exam>

工具包：（用于解析文件，以及将修改的文件写入xml）
package utils;

import java.io.FileOutputStream;

import javax.xml.parsers.DocumentBuilder;
import javax.xml.parsers.DocumentBuilderFactory;
import javax.xml.transform.Transformer;
import javax.xml.transform.TransformerFactory;
import javax.xml.transform.dom.DOMSource;
import javax.xml.transform.stream.StreamResult;

import org.w3c.dom.Document;

public class XmlUtils {

	//xml路径	 
	public static String fileName = "src/exam.xml";
	
	/**
	 * 解析xml文件
	 * @return
	 * @throws Exception
	 */
	public static Document getDocument() throws Exception{
		
		DocumentBuilderFactory factory = DocumentBuilderFactory.newInstance();
		DocumentBuilder builder = factory.newDocumentBuilder();
		return builder.parse(fileName);
	}
	
	/**
	 * 将修改的文件写入xml中
	 * @param document
	 * @throws Exception
	 */
	public static void write2xml(Document document) throws Exception{
		 
		TransformerFactory factory = TransformerFactory.newInstance();
		Transformer transformer = factory.newTransformer();
		transformer.transform(new DOMSource(document), new StreamResult(new FileOutputStream(fileName)));
	}
}

model包：
package model;

public class StudentModel {

	private String idcast;
	private String examid;
	private String name;
	private String location;
	private double grade;
	
	public double getGrade() {
		return grade;
	}
	public void setGrade(double grade) {
		this.grade = grade;
	}
	public String getIdcast() {
		return idcast;
	}
	public void setIdcast(String idcast) {
		this.idcast = idcast;
	}
	public String getExamid() {
		return examid;
	}
	public void setExamid(String examid) {
		this.examid = examid;
	}
	public String getName() {
		return name;
	}
	public void setName(String name) {
		this.name = name;
	}
	public String getLocation() {
		return location;
	}
	public void setLocation(String location) {
		this.location = location;
	}


}

dao包：
package dao;

import org.w3c.dom.Document;

import org.w3c.dom.Element;
import org.w3c.dom.NodeList;

import exception.StudentNotExistException;

import utils.XmlUtils;
import model.StudentModel;

public class StudentDao {

	public void addData(StudentModel s) {
		
		try {
			//解析文件
			Document document = XmlUtils.getDocument();
			
			//创建student元素
			Element studentData = document.createElement("student");
			studentData.setAttribute("idcast", s.getIdcast());
			studentData.setAttribute("examid", s.getExamid());
			
			//创建student的子元素
			Element studentName = document.createElement("name");
			Element studentLocation = document.createElement("location");
			Element studentGrade = document.createElement("grade");
			
			//给子元素添加内容
			studentName.setTextContent(s.getName());
			studentLocation.setTextContent(s.getLocation());
			studentGrade.setTextContent(s.getGrade()+"");
			
			//将子元素放到student下面
			studentData.appendChild(studentName);
			studentData.appendChild(studentLocation);
			studentData.appendChild(studentGrade);
			
			//将student元素放进exam中
			document.getElementsByTagName("exam").item(0).appendChild(studentData);
			
			//更改xml文件
			XmlUtils.write2xml(document);
		} catch (Exception e) {
			throw new RuntimeException(e);
		}
		
		
		
	}
	
	public StudentModel selectData(String examid) {
		
		try {
			//解析xml文件
			Document document = XmlUtils.getDocument();
			
			//获取所有的学生信息
			NodeList list = document.getElementsByTagName("student");
			
			for(int i=0;i<list.getLength();i++){
				Element studentData = (Element)list.item(i);
				//挑选出准开征号相同的ID的学生，输出
				if(studentData.getAttribute("examid").equals(examid)){
					StudentModel student = new StudentModel();
					student.setIdcast(studentData.getAttribute("idcast"));
					student.setExamid(examid);
					student.setName(studentData.getElementsByTagName("name").item(0).getTextContent());
					student.setLocation(studentData.getElementsByTagName("location").item(0).getTextContent());
					student.setGrade(Double.parseDouble(studentData.getElementsByTagName("grade").item(0).getTextContent()));
					return student;
				}
			}	
			return null;
		} catch (Exception e) {
			throw new RuntimeException(e); 
		}

	}
	
	public void deleteData(String name) throws StudentNotExistException {
		
		try {
			//解析xml文件
			Document document = XmlUtils.getDocument();
			
			//挑选出姓名是指定姓名的学生，删除
			NodeList list = document.getElementsByTagName("name");
			for(int i=0;i<list.getLength();i++){
				Element studentName = (Element)list.item(i);
				//删除操作
				if(studentName.getTextContent().equals(name)){
					studentName.getParentNode().getParentNode().removeChild(studentName.getParentNode());
					//更改xml文件
					XmlUtils.write2xml(document);
					return;
				}
			}
			throw new StudentNotExistException(name + "不存在");
		}catch (StudentNotExistException e) {
			throw e ; 
		}
		catch (Exception e) {
			throw new RuntimeException(e); 
		}
		
	}
	
}

测试用的test包：
package test;

import model.StudentModel;

import org.junit.Test;
import dao.StudentDao;
public class xmlTest {

	@Test
	public void testAdd() throws Exception{
		StudentDao dao = new StudentDao();
		StudentModel student = new StudentModel();
		student.setIdcast("993");
		student.setExamid("999");
		student.setName("aaa");
		student.setLocation("天津");
		student.setGrade(21);
		dao.addData(student);
	}
	
	@Test
	public void testSelectData(){
		StudentDao dao = new StudentDao();
		dao.selectData("222");
	}
	
	@Test
	public void testDeleteData() throws Exception{
		StudentDao dao = new StudentDao();
		dao.deleteData("aserfwer");
	}
}

exception包：（删除时加入输入错误数据需要抛出异常）
package exception;

public class StudentNotExistException extends Exception {

	public StudentNotExistException() {
		// TODO Auto-generated constructor stub
	}

	public StudentNotExistException(String message) {
		super(message);
		// TODO Auto-generated constructor stub
	}

	public StudentNotExistException(Throwable cause) {
		super(cause);
		// TODO Auto-generated constructor stub
	}

	public StudentNotExistException(String message, Throwable cause) {
		super(message, cause);
		// TODO Auto-generated constructor stub
	}

}

用于界面交互的UI包：
package UI;

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;

import model.StudentModel;

import dao.StudentDao;
import exception.StudentNotExistException;

public class Main {

	/**
	 * @param args
	 */
	public static void main(String[] args) throws IOException {
		try {
			
			System.out.println("添加学生(a)    删除学生(b)    查找学生(c)");
			System.out.println("请输入操作类型：");
			
			BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
			String type = br.readLine();
			
			if("a".equals(type)){
				
				System.out.println("请输入学生姓名：");
				String name = br.readLine();
				
				System.out.println("请输入学生准考证号：");
				String examid = br.readLine();
				
				System.out.println("请输入学生身份证号：");
				String idcast = br.readLine();
				
				System.out.println("请输入学生所在地：");
				String location = br.readLine();
				
				System.out.println("请输入学生成绩：");
				String grade = br.readLine();
				
				StudentModel s = new StudentModel();
				s.setIdcast(idcast);
				s.setExamid(examid);
				s.setName(name);
				s.setLocation(location);
				s.setGrade(Double.parseDouble(grade));
				
				StudentDao dao = new StudentDao();
				dao.addData(s);
				
				System.out.println("添加成功");
				
			}else if ("b".equals(type)) {
				
				System.out.println("请输入要删除的学生姓名：");
				String name = br.readLine();
				
				try {
					StudentDao dao = new StudentDao();
					dao.deleteData(name);
					System.out.println("删除成功");
				} catch (StudentNotExistException e) {
					System.out.println("您要删除的用户不存在");
				}
				
			}else if ("c".equals(type)) {
				
				System.out.println("请输入要查找的学生准考证号：");
				String idcast = br.readLine();
				
				StudentDao dao = new StudentDao();
				dao.selectData(idcast);
				System.out.println("查找成功");
				
				
			}else {
				System.out.println("不支持您的操作");
			}
			
		} catch (Exception e) {
			e.printStackTrace();
			System.out.println("对不起，出错了");
		}

	}

}
