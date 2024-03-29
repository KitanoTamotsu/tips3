
Google suggest ワークフローを解析してみる
<br>phpはよく知らないのですが、気になってGoogle suggest ワークフローを
<br>動かしながら解析してみました

```
require_once('workflows.php');
$wf = new Workflows();
```
<br>$wfをWorkflowsオブジェクトとしてクリエイトしている感じですね。
<br>workflows.phpには、workflowsオブジェクトのクラスが記述されているのでしょう。
<br>雰囲気を掴みたいだけなので探したりせずにスルーします。

```
$orig = $argv[1];
$xml = $wf->request( "https://suggestqueries.google.com/complete/search?output=toolbar&q=".urlencode( $orig ) );
$xml = simplexml_load_string( utf8_encode($xml) );
```
<br>アルフの受け渡し（argv[1]）からXMLを取得しているようですね。
<br>https://suggestqueries.google.com/complete/search?output=toolbar&q=
<br>このアドレスでgoogle suggestのxmlが取得できるのでしょう。
<br>さっそくターミナルでopenしてみましょう。思った通り下記のxmlが帰ってきました。
<br>ちょっと長いですが全部コピペします。因みにq=alfredとして検索しています

```
<?xml version="1.0"?>
<toplevel>
    <CompleteSuggestion>
        <suggestion data="alfred"/>
    </CompleteSuggestion>
    <CompleteSuggestion>
        <suggestion data="alfred camera"/>
    </CompleteSuggestion>
    <CompleteSuggestion>
        <suggestion data="alfred 使い方"/>
    </CompleteSuggestion>
    <CompleteSuggestion>
        <suggestion data="alfredobannister"/>
    </CompleteSuggestion>
    <CompleteSuggestion>
        <suggestion data="alfred mac"/>
    </CompleteSuggestion>
    <CompleteSuggestion>
        <suggestion data="alfred camera pc"/>
    </CompleteSuggestion>
    <CompleteSuggestion>
        <suggestion data="alfred windows"/>
    </CompleteSuggestion>
    <CompleteSuggestion>
        <suggestion data="alfred dunner"/>
    </CompleteSuggestion>
    <CompleteSuggestion>
        <suggestion data="alfred 意味"/>
    </CompleteSuggestion>
    <CompleteSuggestion>
        <suggestion data="alfredo bannister in"/>
    </CompleteSuggestion>
</toplevel>
```

構造は簡単ですね。dataの中にキーワードと関連する検索ワードをリターンさせて、それが10個列挙されるようです。
なお、suggestqueries.google.comをgoogle.comに変更しても、現状では同じxmlが帰ってきます。

```
$int = 1;
if ($xml) {
	foreach( $xml as $sugg ):
		$data = $sugg->suggestion->attributes()->data;
		$wf->result( $int.'.'.time(), "$data", "$data", 'Search Google for '.$data, 'icon.png'  );
		$int++;
	endforeach;
}
```
<br>リターンされたXMLのsuggestionタグのdata属性の内容を見て、リターンを作成しているようです。

```
$results = $wf->results();
if ( count( $results ) == 0 ):
	$wf->result( 'googlesuggest', $orig, 'No Suggestions', 'No search suggestions found. Search Google for '.$orig, 'icon.png' );
endif;
```
こちらはリターンが何もない時のロジックのようです。まあ無視しましょう。
```
echo $wf->toxml();
```
<br>最後に作成したリターンからAlfred用のXMLを生成後echoしているようです。
<br>今度はアフルのデバッグを使ってみたらAlfred用のXMLをみることができました。
<br>コピペします。
<br>なお、XMLは1文字入力するごとに作成されます。というか、1文字入力するごとに
<br>google suggestが起動されるようです。まぁAlfredがインクリメンタルサーチなので当然か。

```
<items>

 <item uid="1.1613809030" arg="alfred" valid="yes" autocomplete="">
  <title>alfred</title>
  <subtitle>Search Google for alfred</subtitle>
  <icon>icon.png</icon>
 </item>

 <item uid="2.1613809030" arg="alfred camera" valid="yes" autocomplete="">
  <title>alfred camera</title>
  <subtitle>Search Google for alfred camera</subtitle>
  <icon>icon.png</icon>
 </item>

 <item uid="3.1613809030" arg="alfred &#x4F7F;&#x3044;&#x65B9;" valid="yes" autocomplete="">
  <title>alfred &#x4F7F;&#x3044;&#x65B9;</title>
  <subtitle>Search Google for alfred &#x4F7F;&#x3044;&#x65B9;</subtitle>
  <icon>icon.png</icon>
 </item>

 <item uid="4.1613809030" arg="alfredobannister" valid="yes" autocomplete="">
  <title>alfredobannister</title>
  <subtitle>Search Google for alfredobannister</subtitle>
  <icon>icon.png</icon>
 </item>

 <item uid="5.1613809030" arg="alfred mac" valid="yes" autocomplete="">
  <title>alfred mac</title>
  <subtitle>Search Google for alfred mac</subtitle>
  <icon>icon.png</icon>
 </item>

 <item uid="6.1613809030" arg="alfred camera pc" valid="yes" autocomplete="">
  <title>alfred camera pc</title>
  <subtitle>Search Google for alfred camera pc</subtitle>
  <icon>icon.png</icon>
 </item>

 <item uid="7.1613809030" arg="alfred windows" valid="yes" autocomplete="">
  <title>alfred windows</title>
  <subtitle>Search Google for alfred windows</subtitle> 
  <icon>icon.png</icon>
 </item>

 <item uid="8.1613809030" arg="alfred dunner" valid="yes" autocomplete="">
  <title>alfred dunner</title>
  <subtitle>Search Google for alfred dunner</subtitle>
  <icon>icon.png</icon>
 </item>

 <item uid="9.1613809030" arg="alfred &#x610F;&#x5473;" valid="yes" autocomplete="">
  <title>alfred &#x610F;&#x5473;</title>
  <subtitle>Search Google for alfred &#x610F;&#x5473;</subtitle>
  <icon>icon.png</icon>
 </item>

 <item uid="10.1613809030" arg="alfredo bannister in" valid="yes" autocomplete="">
  <title>alfredo bannister in</title>
  <subtitle>Search Google for alfredo bannister in</subtitle>
  <icon>icon.png</icon>
 </item>

</items>
```
<br>確かにtitemタグをみると以下のロジックでセットしていることがよくわかりますね
	

```
result( $int.'.'.time(), "$data", "$data", 'Search Google for '.$data, 'icon.png'  )
```
<br>とすると、Alfred用のXMLをechoすれば、出力インターフェースが表示されるのかな
<br>なんとなく理解できました

<br>
[トップページに戻る](https://kitanotamotsu.github.io/)


