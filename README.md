### 结巴分词器maxcompute版本


该版本是在 [jieba分词器](https://github.com/huaban/jieba-analysis) 基础上改写的。
由于maxcompute的沙箱限制了所有文件读取操作，导致jieba分词器在maxcompute中执行的时候无法读取词典和配置文件。所以我简单的改写了jieba分词器读取资源的方式，把词典和配置文件都改为从maxcompute资源中读取。

如果你打算使用该版本，只需要把配置文件作为资源上传到maxcompute中：
```
    odpscmd$> add file jieba_main.dict;
    odpscmd$> add file jieba_user.dict;
    odpscmd$> add file jieba_prob_emit.txt;
    odpscmd$> add file jieba_stopwords.txt;
```
同样，使用方法和API也需要做一些改变。以下是一个使用jieba分词的UDF示例示例：

-    Demo

``` {.java}
public class JiebaSearchSeg extends UDF {
    JiebaSegmenter jiebaSegmenter = null;
    List<String> stopWords = null;
    public void setup(ExecutionContext ctx){
        try {
            InputStream mainIns = ctx.readResourceFileAsStream("jieba_main.dict"); //主词典，替换成你的主词典资源文件
            InputStream userIns = ctx.readResourceFileAsStream("jieba_user.dict"); //你的自定义词典资源文件
            InputStream emitIns = ctx.readResourceFileAsStream("jieba_prob_emit.txt"); //发射概率，使用jieba内置的
            InputStream stopIns = ctx.readResourceFileAsStream("jieba_stopwords.txt"); //你的自定义停止词资源文件
            jiebaSegmenter = new JiebaSegmenter(mainIns, userIns, emitIns, stopIns);
            stopWords = jiebaSegmenter.getStopWords();
            System.out.println("stopwords.size: " + stopWords.size());
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    public String evaluate(String s) {
        if(StringUtils.isEmpty(s))
            return null;
        List<SegToken> tokenList = jiebaSegmenter.process(s, JiebaSegmenter.SegMode.SEARCH);//分词，分词模式为『搜索』
        List<String> wordList = new ArrayList<String>();
        for(SegToken token : tokenList) {
             wordList.add(token.word);
        }
        return Joiner.on("|").join(wordList);
    }

```
