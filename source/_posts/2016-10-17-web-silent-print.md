---
title: web silent print
date: 2016-10-17 14:05:16
tags:
category: 技术
---

# 应用场景
web 情形下，需要直接打印，不需要预览。
- Chrome自身有一个Kiosk Mode参数，如果设置，可以直接在默认打印机打印，这个没去试，但是应该很快，然后对于如果需要多个和多种打印机，比如打印4x2, 4x6 相关的label，其实不是很合适。
- 早前的解决方式，比如IE的ActiveX，以及NPAPI，但是Chrome目前已经不再支持NPAPI。
- 第三方 lodop(透明格式，非PDF，需要注册)， smartprint(需要注册)
- google cloud(api很完善，但是等待打印时间很长，对于对打印及时性要求不高的情形很适合)
- 打印机自身的控制语言，比如zebra条码打印机，有development相关的控制命令，通过命令可以构建label样式，然后通过直接将命令输出到打印机完成打印，没去细看，估计需要蛮多时间。
- http://jasperreports.sourceforge.net/sample.reference/printservice/， JasperReport(license, LGPL). Java报表系~
- pdfbox + java print，这个是目前通过实验，可以满足基本的打印需求。

# 构想
local通过pdfbox 和 java print构建打印服务，browser 向 远端server请求pdf资源，然后将pdf资源提交给local server，local server 完成打印。目前设备zebra 450，主要用于4X2 inch 的Location打印。

# 部分关键代码

远程server返回pdf资源
```python
with open(obj_replenish_shipment.label_path, 'r') as pdf:
    response_data = HttpResponse(pdf.read(), mimetype='application/pdf')
    response_data['Content-Disposition'] = 'inline;filename=some_file.pdf'
return response_data
```

local 请求远程server资源并转递到local server：
需要注意的这里用到的是XMLHttpRequest去拿binary内容（也就是pdf内容），ajax不行，如果用ajax需要找ajax的扩充。
```javascript
$('#DirectPrint_btn').click(function(e){
    var shelf_id = document.getElementById('print_IDs').value;
    var box_type = $("#box_type_check").is(':checked') ? 'on' : 'off';
    var xhr = new XMLHttpRequest();
    xhr.open("GET", "/print_labels?ids="+shelf_id + "&box_type_check="+box_type, true);
    xhr.responseType = 'arraybuffer';
    xhr.onload = function(e) {
        var responseArray = new Uint8Array(this.response);
        var blob = new Blob([responseArray], {type: 'application/pdf'});
//      var fileURL = URL.createObjectURL(blob);
//      window.open(fileURL);
        var formData = new FormData();
        formData.append("why", "REQUEST_PRINT");
        params = [{
            "type": "Location",
            "sub_type": "t1(4x2)",
            "file_name": "xx.pdf",
            "page_number_range": "all",
            "count": "2"
        }];
        formData.append("params", JSON.stringify(params));
        formData.append("file", blob, "xx.pdf"); //file one by one
        
        var request = new XMLHttpRequest();
        request.open("POST", "http://localhost:8080/LocalPrinter/service.jsp", true);
        request.send(formData);
    }
    xhr.send();
});
```

local server使用apache tomcat搭建，关键部分，zebra 450驱动装好，连接后需要做setting，有两个部分的setting都需要做。
![默认setting](/img/zebra_default_setting_s1.png "--")
![首选项](/img/zebra_first_setting_s1.png "--")

payload 的parse用的是com.oreilly.servlet.multipart 这个包

```java
if(node.position.equals("localhost")) {
    try {
        PrintService  printService = null;
        PrintService[] pServices = PrintServiceLookup.lookupPrintServices(null, null);
        if(pServices.length == 0) {
            System.out.println("No print service found.");
            return;
        }
        
        //define output stream
        FileOutputStream _o = null;
        _o = new FileOutputStream(_path + "out.pdf");
        
        //list printer
        if (pServices != null){
            System.out.println("listing printers " + pServices.length);
            
            for (int i=0;i<pServices.length;i++){
                printService = pServices[i];
                System.out.println(pServices[i]);
            }
            System.out.println("listing end~");
        }
        
        //find printer
        boolean find = false;
        for (int i = 0; i < pServices.length; i++) {
            if(pServices[i].getName().equals(node.printer_name)) {
                System.out.println("*************************************************************");
                System.out.println("find printer: " + node.printer_name);
                System.out.println("**************************************************************");
                printService = pServices[i];
                find = true;
                break;
            }
        }
        if(!find) return;
        
        PDDocument document = null;
        document = PDDocument.load(p.file_content);
        document.save(new File(_path + "xx.pdf"));
        
        PrinterJob job = PrinterJob.getPrinterJob();
        job.setPrintService(printService);
        
        //custom format & paper
        PageFormat pf = job.defaultPage();
        Paper paper = pf.getPaper();
        paper.setImageableArea(0, 0, paper.getWidth(), paper.getHeight());
        pf.setPaper(paper);
        pf.setOrientation(PageFormat.PORTRAIT);
        job.defaultPage(pf);
        
        //Book
        Book book = new Book();
        book.append(new PDFPrintable(document), pf, document.getNumberOfPages());
        
        job.setPageable(book);
        
        PrintRequestAttributeSet job_attrs = new HashPrintRequestAttributeSet();
        if(node.type.equals("Location") && node.sub_type.equals("t1(4x2)")) {
            //job_attrs.add(new MediaSize(2.00f, 4.00f, Size2DSyntax.INCH));
            job_attrs.add(new MediaPrintableArea(0, 0, 4, 2, MediaPrintableArea.INCH));
        }
        
        job.print(job_attrs);
        System.out.print("Pushing job");
        
        document.close();
        _o.close();
    } catch (Exception e) {
        e.printStackTrace();
    }
    
    System.out.println("Printing launched, please wait...");
}
```

