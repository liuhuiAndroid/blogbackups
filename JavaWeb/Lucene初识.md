###什么是全文检索，如何实现全文检索
先建立索引，再对索引进行搜索的过程就叫**全文检索**(**Full-text Search**)。
可以使用Lucene实现全文检索。Lucene是apache下的一个开放源代码的全文检索引擎工具包。提供了完整的查询引擎和索引引擎，部分文本分析引擎。

###[Lucene](http://lucene.apache.org/)实现全文检索的流程
* 创建文档对象
将原始内容创建为包含域（Field）的文档（Document）
* 分析文档
对原始文档分析生成语汇单元
* 创建索引
对所有文档分析得出的语汇单元进行索引，索引的目的是为了搜索，最终要实现只搜索被索引的语汇单元从而找到文档（Document）。
创建索引是对语汇单元索引，通过词语或内容找文档，这种索引的结构叫**倒排索引结构**。
**倒排索引结构也叫反向索引结构，包括索引和文档两部分，索引即词汇表，它的规模较小，而文档集合较大。**
* 查询索引
查询索引也是搜索的过程。搜索就是用户输入关键字，从索引中进行搜索的过程。根据关键字搜索索引，根据索引找到对应的文档，从而找到要搜索的内容。

###配置[Lucene](http://lucene.apache.org/)开发环境
* 需要用到的jar包
lucene-core-4.10.3.jar
lucene-analyzers-common-4.10.3.jar
lucene-queryparser-4.10.3.jar
以及其他包：
commons-io-2.4.jar
junit-4.9.jar

###创建索引库
使用indexwriter对象**创建索引**

是否分析：是否对域的内容进行分词处理。前提是我们要对域的内容进行查询。
是否索引：将Field分析后的词或整个Field值进行索引，只有索引方可搜索到。
比如：商品名称、商品简介分析后进行索引，订单号、身份证号不用分析但也要索引，这些将来都要作为查询条件。
是否存储：将Field值存储在文档中，存储在文档中的Field才可以从Document中获取
比如：商品名称、订单号，凡是将来要从Document中获取的Field都要存储。
**是否存储的标准：是否要将内容展示给用户**

![image.png](https://upload-images.jianshu.io/upload_images/1956963-83e4eae958c94f5e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```
	@Test
	public void testIndex() throws Exception {
		// 第一步：创建一个java工程，并导入jar包。
		// 第二步：创建一个indexwriter对象。
		Directory directory = FSDirectory.open(new File("D:\\temp\\index"));
		// Directory directory = new RAMDirectory();//保存索引到内存中 （内存索引库）
		// Analyzer analyzer = new StandardAnalyzer();// 官方推荐
		Analyzer analyzer = new IKAnalyzer();// 官方推荐
		IndexWriterConfig config = new IndexWriterConfig(Version.LATEST, analyzer);
		// 1）指定索引库的存放位置Directory对象
		// 2）指定一个分析器，对文档内容进行分析。
		IndexWriter indexWriter = new IndexWriter(directory, config);
		// 第四步：创建field对象，将field添加到document对象中。
		File f = new File("D:\\Lucene&solr\\searchsource");
		File[] listFiles = f.listFiles();
		for (File file : listFiles) {
			// 第三步：创建document对象。
			Document document = new Document();
			// 文件名称
			String file_name = file.getName();
			Field fileNameField = new TextField("fileName", file_name, Store.YES);
			// 文件大小
			long file_size = FileUtils.sizeOf(file);
			Field fileSizeField = new LongField("fileSize", file_size, Store.YES);
			// 文件路径
			String file_path = file.getPath();
			Field filePathField = new StoredField("filePath", file_path);
			// 文件内容
			String file_content = FileUtils.readFileToString(file);
			Field fileContentField = new TextField("fileContent", file_content, Store.NO);
			document.add(fileNameField);
			document.add(fileSizeField);
			document.add(filePathField);
			document.add(fileContentField);
			// 第五步：使用indexwriter对象将document对象写入索引库，此过程进行索引创建。并将索引和document对象写入索引库。
			indexWriter.addDocument(document);
		}
		// 第六步：关闭IndexWriter对象。
		indexWriter.close();
	}
```
* 可以使用Luke工具查看索引文件
[**lukeall.jar**](http://www.getopt.org/luke/luke-0.9.9/lukeall-0.9.9.jar)

###查询索引库
```
	// 搜索索引
	@Test
	public void testSearch() throws Exception {
		// 第一步：创建一个Directory对象，也就是索引库存放的位置。
		Directory directory = FSDirectory.open(new File("D:\\temp\\index"));// 磁盘
		// 第二步：创建一个indexReader对象，需要指定Directory对象。
		IndexReader indexReader = DirectoryReader.open(directory);
		// 第三步：创建一个indexsearcher对象，需要指定IndexReader对象
		IndexSearcher indexSearcher = new IndexSearcher(indexReader);
		// 第四步：创建一个TermQuery对象，指定查询的域和查询的关键词。
		Query query = new TermQuery(new Term("fileName", "lucene"));
		// 第五步：执行查询。
		TopDocs topDocs = indexSearcher.search(query, 10);
		// 第六步：返回查询结果。遍历查询结果并输出。
		ScoreDoc[] scoreDocs = topDocs.scoreDocs;
		for (ScoreDoc scoreDoc : scoreDocs) {
			int doc = scoreDoc.doc;
			Document document = indexSearcher.doc(doc);
			// 文件名称
			String fileName = document.get("fileName");
			System.out.println(fileName);
			// 文件内容
			String fileContent = document.get("fileContent");
			System.out.println(fileContent);
			// 文件大小
			String fileSize = document.get("fileSize");
			System.out.println(fileSize);
			// 文件路径
			String filePath = document.get("filePath");
			System.out.println(filePath);
			System.out.println("------------");
		}
		// 第七步：关闭IndexReader对象
		indexReader.close();
	}
```
####TopDocs
Lucene搜索结果可通过TopDocs遍历。TopDocs的属性：

| 方法或属性| 说明 | 备注| 
| - | :-: | -: | 
| totalHits | 匹配搜索条件的总记录数| 0 | 
| scoreDocs  | 顶部匹配记录 | 0 | 

注意：
Search方法需要指定匹配记录数量n：indexSearcher.search(query, n)
TopDocs.totalHits：是匹配索引库中所有记录的数量
TopDocs.scoreDocs：匹配相关度高的前边记录数组，scoreDocs的长度小于等于search方法指定的参数n

###分析器的分析过程
* 测试分析器的分词效果
```
	@Test
	public void testTokenStream() throws Exception {
		// 创建一个标准分析器对象
		// Analyzer analyzer = new StandardAnalyzer();
		// Analyzer analyzer = new CJKAnalyzer();
		// Analyzer analyzer = new SmartChineseAnalyzer();
		Analyzer analyzer = new IKAnalyzer();
		// 获得tokenStream对象
		// 第一个参数：域名，可以随便给一个
		// 第二个参数：要分析的文本内容
		// TokenStream tokenStream = analyzer.tokenStream("test",
		// The Spring Framework provides a comprehensive programming and configuration model.");
		TokenStream tokenStream = analyzer.tokenStream("test","燕子去了，有再来的时候。");
		// 添加一个引用，可以获得每个关键词
		CharTermAttribute charTermAttribute = tokenStream.addAttribute(CharTermAttribute.class);
		// 添加一个偏移量的引用，记录了关键词的开始位置以及结束位置
		OffsetAttribute offsetAttribute = tokenStream.addAttribute(OffsetAttribute.class);
		// 将指针调整到列表的头部
		tokenStream.reset();
		// 遍历关键词列表，通过incrementToken方法判断列表是否结束
		while (tokenStream.incrementToken()) {
			// 关键词的起始位置
			System.out.println("start->" + offsetAttribute.startOffset());
			// 取关键词
			System.out.println(charTermAttribute);
			// 结束位置
			System.out.println("end->" + offsetAttribute.endOffset());
		}
		tokenStream.close();
	}
```
* **IKAnalyzer中文分词器**
1. 添加 IKAnalyzer2012FF_u1.jar
2. 把配置文件 IKAnalyzer.cfg.xml 和扩展词典和停用词词典添加到classpath下
```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE properties SYSTEM "http://java.sun.com/dtd/properties.dtd">  
<properties>  
	<comment>IK Analyzer 扩展配置</comment>
	<!--用户可以在这里配置自己的扩展字典 -->
	<entry key="ext_dict">ext.dic;</entry> 
	<!--用户可以在这里配置自己的扩展停止词字典-->
	<entry key="ext_stopwords">stopword.dic;</entry> 
</properties>
```
注意扩展词典和停用词词典都是无BOM 的UTF-8 编码。

**索引时和搜索时可以使用Analyzer，注意搜索使用的分析器要和索引使用的分析器一致。**

###索引库的维护
* 添加文档
```
	//创建IndexWriter对象
	public IndexWriter getIndexWriter() throws Exception{
		// 第一步：创建一个java工程，并导入jar包。
		// 第二步：创建一个indexwriter对象。
		Directory directory = FSDirectory.open(new File("D:\\temp\\index"));
		// Directory directory = new RAMDirectory();//保存索引到内存中 （内存索引库）
		Analyzer analyzer = new StandardAnalyzer();// 官方推荐
		IndexWriterConfig config = new IndexWriterConfig(Version.LATEST, analyzer);
		return new IndexWriter(directory, config);
	}
	
	//添加索引
	@Test
	public void addDocument() throws Exception {
		IndexWriter indexWriter = getIndexWriter();
		//创建一个Document对象
		Document doc = new Document();
		doc.add(new TextField("fileName", "测试文件名",Store.YES));
		doc.add(new TextField("fileContent", "测试文件内容",Store.YES));
		//添加文档到索引库
		indexWriter.addDocument(doc);
		//关闭IndexWriter
		indexWriter.close();
	}
```
* 删除文档
```
	//全删除 慎用
	@Test
	public void testAllDelete() throws Exception {
		IndexWriter indexWriter = getIndexWriter();
		indexWriter.deleteAll();
		indexWriter.close();
	}
	
	//根据条件删除
	@Test
	public void testDelete() throws Exception {
		IndexWriter indexWriter = getIndexWriter();
		Query query = new TermQuery(new Term("fileName","apache"));
		indexWriter.deleteDocuments(query);
		indexWriter.close();
	}
```

* 修改文档
```
	//修改
	@Test
	public void testUpdate() throws Exception {
		IndexWriter indexWriter = getIndexWriter();
		Document doc = new Document();
		doc.add(new TextField("fileN", "测试文件名",Store.YES));
		doc.add(new TextField("fileC", "测试文件内容",Store.YES));
		indexWriter.updateDocument(new Term("fileName","lucene"), doc, new IKAnalyzer());
		indexWriter.close();
	}
```

###[Lucene](http://lucene.apache.org/)的高级查询
1. 使用Query的子类查询
* MatchAllDocsQuery
使用MatchAllDocsQuery查询索引目录中的所有文档
```
	//创建IndexSearcher对象  
	public IndexSearcher getIndexSearcher() throws Exception{
		// 第一步：创建一个Directory对象，也就是索引库存放的位置。
		Directory directory = FSDirectory.open(new File("D:\\temp\\index"));// 磁盘
		// 第二步：创建一个indexReader对象，需要指定Directory对象。
		IndexReader indexReader = DirectoryReader.open(directory);
		// 第三步：创建一个indexsearcher对象，需要指定IndexReader对象
		return new IndexSearcher(indexReader);
	}
	
	//查询所有
	@Test
	public void testMatchAllDocsQuery() throws Exception {
		IndexSearcher indexSearcher = getIndexSearcher();
		Query query = new MatchAllDocsQuery();
		System.out.println(query);
		printResult(indexSearcher, query);
		//关闭资源
		indexSearcher.getIndexReader().close();
	}
	
	//执行查询的结果
	public void printResult(IndexSearcher indexSearcher,Query query)throws Exception{
		// 第五步：执行查询。
		TopDocs topDocs = indexSearcher.search(query, 10);
		// 第六步：返回查询结果。遍历查询结果并输出。
		ScoreDoc[] scoreDocs = topDocs.scoreDocs;
		for (ScoreDoc scoreDoc : scoreDocs) {
			int doc = scoreDoc.doc;
			Document document = indexSearcher.doc(doc);
			// 文件名称
			String fileName = document.get("fileName");
			System.out.println(fileName);
			// 文件内容
			String fileContent = document.get("fileContent");
			System.out.println(fileContent);
			// 文件大小
			String fileSize = document.get("fileSize");
			System.out.println(fileSize);
			// 文件路径
			String filePath = document.get("filePath");
			System.out.println(filePath);
			System.out.println("------------");
		}
	}
```
* TermQuery
TermQuery不使用分析器所以建议匹配不分词的Field域查询，比如订单号、分类ID号等。
指定要查询的域和要查询的关键词。
```
	//使用Termquery查询
	@Test
	public void testTermQuery() throws Exception {
		IndexSearcher indexSearcher = getIndexSearcher();
		//创建查询对象
		Query query = new TermQuery(new Term("fileName", "mybatis.txt"));
		//执行查询
		TopDocs topDocs = indexSearcher.search(query, 10);
		//共查询到的document个数
		System.out.println("查询结果总数量：" + topDocs.totalHits);
		//遍历查询结果
		printResult(indexSearcher, query);
		//关闭indexreader
		indexSearcher.getIndexReader().close();
	}
```
* NumericRangeQuery
可以根据数值范围查询。
```
	//根据数值范围查询
	@Test
	public void testNumericRangeQuery() throws Exception {
		IndexSearcher indexSearcher = getIndexSearcher();
		
		Query query = NumericRangeQuery.newLongRange("fileSize", 47L, 200L, false, true);
		System.out.println(query);
		printResult(indexSearcher, query);
		//关闭资源
		indexSearcher.getIndexReader().close();
	}
```
* BooleanQuery
可以组合查询条件。
```
	//可以组合查询条件
	@Test
	public void testBooleanQuery() throws Exception {
		IndexSearcher indexSearcher = getIndexSearcher();
		BooleanQuery booleanQuery = new BooleanQuery();
		Query query1 = new TermQuery(new Term("fileName","spring"));
		Query query2 = new TermQuery(new Term("fileName","web"));
		//  select * from user where id =1 or name = 'safdsa'
		booleanQuery.add(query1, Occur.MUST);
		booleanQuery.add(query2, Occur.SHOULD);
		System.out.println(booleanQuery);
		printResult(indexSearcher, booleanQuery);
		//关闭资源
		indexSearcher.getIndexReader().close();
	}
```

2. 使用QueryParser查询
通过QueryParser也可以创建Query，QueryParser提供一个Parse方法，此方法可以直接根据查询语法来查询。
* QueryParser
需要加入queryParser依赖的jar包：lucene-queryparser-4.10.3.jar
```
	//条件解释的对象查询
	@Test
	public void testQueryParser() throws Exception {
		IndexSearcher indexSearcher = getIndexSearcher();
		//参数1： 默认查询的域  
		//参数2：采用的分析器
		QueryParser queryParser = new QueryParser("fileName",new IKAnalyzer());
		// *:*   域：值
		Query query = queryParser.parse("fileName:lucene is apache OR fileContent:lucene is apache");
		printResult(indexSearcher, query);
		//关闭资源
		indexSearcher.getIndexReader().close();
	}
```
* MultiFieldQueryParser
可以指定多个默认搜索域
```
	//条件解析的对象查询   多个默念域
	@Test
	public void testMultiFieldQueryParser() throws Exception {
		IndexSearcher indexSearcher = getIndexSearcher();
		String[] fields = {"fileName","fileContent"};
		//参数1： 默认查询的域  
		//参数2：采用的分析器
		MultiFieldQueryParser queryParser = new MultiFieldQueryParser(fields,new IKAnalyzer());
		// *:*   域：值
		Query query = queryParser.parse("lucene is apache");
		printResult(indexSearcher, query);
		//关闭资源
		indexSearcher.getIndexReader().close();
	}
```
