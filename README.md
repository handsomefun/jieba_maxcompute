结巴分词(odps版) jieba-odps
===============================
该版本是再jieba分词器Java版基础上改写的，由于odps的沙箱限制导致jieba分词器在odps中无法读取词典和配置文件。
所以我们简单的改写了一下jieba分词器读取资源的方式，把词典和配置文件都改为从odps资源中读取，同时把这些配置作为资源上传到odps中。
同样，使用方式也需要做一些改变。

-    Demo

``` {.java}


public class JiebaSearchSeg extends UDF {
    JiebaSegmenter jiebaSegmenter = null;
    List<String> stopWords = null;
    public void setup(ExecutionContext ctx){
        try {
            InputStream mainIns = ctx.readResourceFileAsStream("jieba_main.dict");
            InputStream userIns = ctx.readResourceFileAsStream("jieba_user.dict");
            InputStream emitIns = ctx.readResourceFileAsStream("jieba_prob_emit.txt");
            InputStream stopIns = ctx.readResourceFileAsStream("jieba_stopwords.txt");
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
        List<SegToken> tokenList = jiebaSegmenter.process(s, JiebaSegmenter.SegMode.SEARCH);
        List<String> wordList = new ArrayList<String>();
        for(SegToken token : tokenList) {
            if(!stopWords.contains(token.word) && !StringTools.isMatchPunct(token.word))
                wordList.add(token.word);
        }
        return Joiner.on("|").join(wordList);
    }

}

```