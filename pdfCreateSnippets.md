# 1. PDF Create Snippets [Reference](https://www.baeldung.com/java-pdf-creation)

```java
	public String writePdf() {
		
		Document document = new Document();
		try {
			PdfWriter.getInstance(document, new FileOutputStream("iTextHelloWorld.pdf"));
			document.open();
			Font font = FontFactory.getFont(FontFactory.COURIER, 16, BaseColor.BLACK);
			Chunk chunk = new Chunk("Hello World", font);
			Paragraph para = new Paragraph();
			para.add("Paragraph String.");
			document.add(chunk);
			document.add(para);
			document.close();			
			
			Path path = Paths.get("C:\\Users\\...\\Downloads\\dinosaur.png");

			document = new Document();
			PdfWriter.getInstance(document, new FileOutputStream("iTextImageExample.pdf"));
			document.open();
			
			
			Image img = Image.getInstance(path.toAbsolutePath().toString());
			document.add(img);

			document.close();
			
			document = new Document();
			PdfWriter.getInstance(document, new FileOutputStream("iTextTable.pdf"));

			document.open();

			PdfPTable table = new PdfPTable(3);
			addTableHeader(table);
			addRows(table);
			addCustomRows(table);

			document.add(table);
			document.close();
			
			/*PDF Box Document*/
			PDDocument document1 = new PDDocument();
			PDPage page = new PDPage();
			document1.addPage(page);

			PDPageContentStream contentStream = new PDPageContentStream(document1, page);

			contentStream.setFont(PDType1Font.COURIER, 12);
			contentStream.beginText();
			contentStream.showText("Hello World");
			contentStream.endText();
			contentStream.close();

			document1.save("pdfBoxHelloWorld.pdf");
			document1.close();
			
			/*PDF Box Pictures*/
			PDDocument document2 = new PDDocument();
			PDPage page2 = new PDPage();
			document2.addPage(page2);

			Path path2 = Paths.get("C:\\Users\\...\\Downloads\\dinosaur.png");
			PDPageContentStream contentStream2 = new PDPageContentStream(document2, page2);
			PDImageXObject image 
			  = PDImageXObject.createFromFile(path2.toAbsolutePath().toString(), document2);
			contentStream2.drawImage(image, 0, 0);
			contentStream2.close();

			document2.save("pdfBoxImage.pdf");
			document2.close();
			
		} catch (FileNotFoundException e) {
			e.printStackTrace();
		} catch (DocumentException e) {
			e.printStackTrace();
		} catch (IOException e) {
			e.printStackTrace();
		}		
		
		return "Write Successful!";
	}
```

# 2. Dependencies 
```xml
<dependency>
    <groupId>com.itextpdf</groupId>
    <artifactId>itextpdf</artifactId>
    <version>5.5.13.3</version>
</dependency>
<dependency>
    <groupId>org.apache.pdfbox</groupId>
    <artifactId>pdfbox</artifactId>
    <version>3.0.0-RC1</version>
</dependency>
<dependency>
    <groupId>org.bouncycastle</groupId>
    <artifactId>bcprov-jdk15on</artifactId>
    <version>1.56</version>
</dependency>
```
