---
title: "Handling Excel-Files (*.xls, *.xlsx) in Web Applications ICSharpCode.SharpZipLib.Zip.ZipException: EOF in header"
tags:
  - NPOI
  - C#
  - ASP.NET
  - Excel
  - SharpZipLib
---

I'm working on a SaaS application in ASP.NET MVC, which takes your [KPIs](http://en.wikipedia.org/wiki/Performance_indicator) and presents them to you in clear overviews and graphs, so you can see at a glance, how your business is doing and what needs to change. This is for [SMEs](http://en.wikipedia.org/wiki/Small_and_medium-sized_enterprises), which to a large extent use Excel for these kind of solutions. A large part of the application is, allowing the customers to import the data easily. As most of the data already exist in Excel sheets, we thought it would be the easiest, to just let them upload those.

My first attempt was [Office Interop](https://msdn.microsoft.com/en-us/library/microsoft.office.interop.excel%28v=office.15%29.aspx). However, this does not work on a server. It used to in older versions, but even back then Microsoft was saying, you should not do it, and now it got even harder, so do yourself a favor and don't try this at home.

The second attempt wasn't much better with [Open-XML-SDK](https://github.com/OfficeDev/Open-XML-SDK). This felt like not just building your own computer, but building your own CPU. Way too complicated for Excel sheets (I'm generating Word documents with it, and that works fine.), and it has the additional disadvantage, that the users first need to convert their old files into the new `xlsx` format.

But sure there must be something, even Java can do it with [POI](https://poi.apache.org/), so why not .Net? Well, turns out, there is [NPOI](https://github.com/tonyqus/npoi) and it's awesome. It has support for new and old Excel files (`xlsx`, `xls`), is really easy to use and can even evaluate formulas. So I implemented it and everything was great, until - few months later, I realized, `xlsx` files aren't working. They fail with the following error:

    Type: "ICSharpCode.SharpZipLib.Zip.ZipException"
    Message: "EOF in header"
    StackTrace: at ICSharpCode.SharpZipLib.Zip.Compression.Streams.InflaterInputBuffer.ReadLeByte () [0x00000] in <filename unknown>:0
                at ICSharpCode.SharpZipLib.Zip.Compression.Streams.InflaterInputBuffer.ReadLeShort () [0x00000] in <filename unknown>:0
                at ICSharpCode.SharpZipLib.Zip.Compression.Streams.InflaterInputBuffer.ReadLeInt () [0x00000] in <filename unknown>:0
                at ICSharpCode.SharpZipLib.Zip.ZipInputStream.GetNextEntry () [0x00000] in <filename unknown>:0
                at (wrapper remoting-invoke-with-check) ICSharpCode.SharpZipLib.Zip.ZipInputStream:GetNextEntry ()
                at NPOI.OpenXml4Net.Util.ZipInputStreamZipEntrySource..ctor (ICSharpCode.SharpZipLib.Zip.ZipInputStream inp) [0x00000] in <filename unknown>:0
                at NPOI.OpenXml4Net.OPC.ZipPackage..ctor (System.IO.Stream in1, PackageAccess access) [0x00000] in <filename unknown>:0
                at NPOI.OpenXml4Net.OPC.OPCPackage.Open (System.IO.Stream in1) [0x00000] in <filename unknown>:0
                at NPOI.Util.PackageHelper.Open (System.IO.Stream is1) [0x00000] in <filename unknown>:0
                at NPOI.XSSF.UserModel.XSSFWorkbook..ctor (System.IO.Stream is1) [0x00000] in <filename unknown>:0
                ...

Searching for this problem online was for me somehow confusing. Generally it was suggested, to reset the position of the stream back to the beginning, but I never changed the position of the stream - weird. Let's look at the code and the simple solution. I'm using the default suggestion to handle file uploads, so that these files wouldn't end up on my server disc:

    [HttpPost]
    public void UploadData()
    {
        var postedFiles = HttpContext.Current.Request.Files;

        for (int i = 0; i < postedFiles.Count; i++)
        {
            var file = postedFiles[i];
            var fileName = file.FileName.ToLower();

            using (MemoryStream memstream = new MemoryStream())
            {
                file.InputStream.CopyTo(memstream);

                if (fileName.EndsWith(".xlsx"))
                {
                    UploadData(new XSSFWorkbook(memstream));
                }
                else if (fileName.EndsWith(".xls"))
                {
                    UploadData(new HSSFWorkbook(memstream));
                }
            }
        }
    }

    private UploadData(IWorkbook wb) { ... }

Well, turns out, `Stream.CopyTo(Stream)` is not some magic function, it just goes through the `InputStream` and writes it byte-by-byte into `memstream`. In the end, the cursor is obviously at the last position. Because of the way `SharpZipLib` is implemented, to be more efficient and not require seeks, it just starts reading at the current position and ends rapidly, because it is the end. So the fix is adding a simple `memstream.Position = 0;` and it works, here the whole code one more time:

    [HttpPost]
    public void UploadData()
    {
        var postedFiles = HttpContext.Current.Request.Files;

        for (int i = 0; i < postedFiles.Count; i++)
        {
            var file = postedFiles[i];
            var fileName = file.FileName.ToLower();

            using (MemoryStream memstream = new MemoryStream())
            {
                file.InputStream.CopyTo(memstream);
                memstream.Position = 0; // <-- Add this, to make it work

                if (fileName.EndsWith(".xlsx"))
                {
                    UploadData(new XSSFWorkbook(memstream));
                }
                else if (fileName.EndsWith(".xls"))
                {
                    UploadData(new HSSFWorkbook(memstream));
                }
            }
        }
    }

    private UploadData(IWorkbook wb) { ... }
