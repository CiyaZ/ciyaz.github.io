# Lucene-全文检索引擎

Lucene是一个应用非常广泛的开源全文检索框架。

## 应用场景

我们日常开发软件中，经常碰到在一大堆文件中搜索内容的需求，比如我们要在`ciyaz.github.io`的所有文档中搜索`协程`这个关键字，找出和该关键字最相关的前50条记录，这如何实现呢？最垃圾的办法就是遍历所有文档，或者是类似数据库中进行`like`操作，但这显然也是非常低效的。文档数很多时，这几乎不可能实现。

比较好的办法就是对文档分词后进行倒排索引，调用现成的分词字典库，然后记录下词语出现的文档序号作为索引，我们按关键字查询时，就能很快得到包含该词的文档了。但实际需求可能复杂得多，比如需要支持更复杂的查询表达式，需要根据某些特定规则排序，需要支持分页等等，一个真正的搜索引擎，其实是相当复杂的！好在有Lucene，把这些全都集成好了，我们直接使用就行了。

## 例子

下面例子代码中，我们实现了一个简单的功能，用Lucene为我的笔记仓库`ciyaz.github.io`中所有文本建立索引，然后搜索`协程`这个关键字。

```java
package com.ciyaz.demo;

import org.apache.lucene.analysis.Analyzer;
import org.apache.lucene.document.Document;
import org.apache.lucene.document.Field;
import org.apache.lucene.document.TextField;
import org.apache.lucene.index.DirectoryReader;
import org.apache.lucene.index.IndexWriter;
import org.apache.lucene.index.IndexWriterConfig;
import org.apache.lucene.queryparser.classic.ParseException;
import org.apache.lucene.queryparser.classic.QueryParser;
import org.apache.lucene.search.IndexSearcher;
import org.apache.lucene.search.Query;
import org.apache.lucene.search.ScoreDoc;
import org.apache.lucene.search.TopDocs;
import org.apache.lucene.store.Directory;
import org.apache.lucene.store.FSDirectory;
import org.apache.lucene.util.Version;
import org.wltea.analyzer.lucene.IKAnalyzer;

import java.io.*;

/**
 * @author ciyaz
 */
public class Main {
    public static void main(String[] args) {
        Main main = new Main();

        System.out.println("开始建立索引...");
        try {
            main.makeIndex();
        } catch (IOException e) {
            e.printStackTrace();
        }
        System.out.println("建立索引完成");

        System.out.println("开始搜索...");
        try {
            main.search("协程");
        } catch (IOException | ParseException e) {
            e.printStackTrace();
        }
        System.out.println("搜索完成");
    }

    /**
     * 建立索引
     */
    public void makeIndex() throws IOException {

        IndexWriter writer;

        // FSDirectory 即使用文件系统存储，这里指定了存储路径
        Directory directory = FSDirectory.open(new File("E:/indexer"));
        // IK-Analyzer是一个开源的中文分词器
        Analyzer analyzer = new IKAnalyzer();
        // IndexWriter 索引写入工具类
        IndexWriterConfig config = new IndexWriterConfig(Version.LUCENE_4_10_4, analyzer);
        writer = new IndexWriter(directory, config);

        // 读取所有文档，用于生成索引，具体目录结构的读取逻辑可以忽略
        File docCatalogFilesRoot = new File("E:/ciyaz.github.io");
        File[] docCatalogFiles = docCatalogFilesRoot.listFiles();
        if (docCatalogFiles != null) {
            for (File docCatalogFile : docCatalogFiles) {
                if (docCatalogFile.isDirectory()) {
                    File[] docTopicFiles = docCatalogFile.listFiles();
                    if (docTopicFiles != null) {
                        for (File docTopicFile : docTopicFiles) {
                            if (docTopicFile.isDirectory()) {
                                File[] docFiles = docTopicFile.listFiles();
                                if (docFiles != null) {
                                    for (File docFile : docFiles) {
                                        if (docFile.isDirectory()) {
                                            String docTitle = docFile.getName();
                                            String docPath = docFile.getAbsolutePath();
                                            File[] docResources = docFile.listFiles();
                                            if (docResources != null) {
                                                for (File docResource : docResources) {
                                                    if (docResource.getName().endsWith(".md")) {
                                                        InputStream is = new FileInputStream(docResource);
                                                        InputStreamReader isr = new InputStreamReader(is);
                                                        BufferedReader docContentReader = new BufferedReader(isr);
                                                        // 创建标准的Document对象并将其信息写入索引，title、path、content是其三个字段
                                                        Document document = new Document();
                                                        document.add(new TextField("title", docTitle, Field.Store.YES));
                                                        document.add(new TextField("path", docPath, Field.Store.YES));
                                                        document.add(new TextField("content", docContentReader));
                                                        writer.addDocument(document);
                                                        break;
                                                    }
                                                }
                                            }
                                        }
                                    }
                                }
                            }
                        }
                    }
                }
            }
        }
        writer.close();
    }

    /**
     * 进行搜索
     *
     * @param keyword 关键字
     */
    public void search(String keyword) throws IOException, ParseException {
        Directory directory = FSDirectory.open(new File("E:/indexer"));
        // IndexSearcher 索引搜索工具类
        IndexSearcher searcher = new IndexSearcher(DirectoryReader.open(directory));
        Analyzer analyzer = new IKAnalyzer();
        // QueryParser 查询工具类，构造函数指定了查询的文档字段和分词器
        QueryParser queryParser = new QueryParser("content", analyzer);
        // 进行搜索
        Query query = queryParser.parse(keyword);
        TopDocs topDocs = searcher.search(query, 50);
        // 输出搜索结果
        for (ScoreDoc scoreDoc : topDocs.scoreDocs) {
            Document document = searcher.doc(scoreDoc.doc);
            System.out.println(document.get("path"));
        }
    }
}
```
