
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
&lt?xml version="1.0"?&gt
&lttoplevel&gt
    &ltCompleteSuggestion&gt
        &ltsuggestion data="alfred"/&gt
    &lt/CompleteSuggestion&gt
    &ltCompleteSuggestion&gt
        &ltsuggestion data="alfred camera"/&gt
    &lt/CompleteSuggestion&gt
    &ltCompleteSuggestion&gt
        &ltsuggestion data="alfred 使い方"/&gt
    &lt/CompleteSuggestion&gt
    &ltCompleteSuggestion&gt
        &ltsuggestion data="alfredobannister"/&gt
    &lt/CompleteSuggestion&gt
    &ltCompleteSuggestion&gt
        &ltsuggestion data="alfred mac"/&gt
    &lt/CompleteSuggestion&gt
    &ltCompleteSuggestion&gt
        &ltsuggestion data="alfred camera pc"/&gt
    &lt/CompleteSuggestion&gt
    &ltCompleteSuggestion&gt
        &ltsuggestion data="alfred windows"/&gt
    &lt/CompleteSuggestion&gt
    &ltCompleteSuggestion&gt
        &ltsuggestion data="alfred dunner"/&gt
    &lt/CompleteSuggestion&gt
    &ltCompleteSuggestion&gt
        &ltsuggestion data="alfred 意味"/&gt
    &lt/CompleteSuggestion&gt
    &ltCompleteSuggestion&gt
        &ltsuggestion data="alfredo bannister in"/&gt
    &lt/CompleteSuggestion&gt
&lt/toplevel&gt
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
&ltitems&gt

 &ltitem uid="1.1613809030" arg="alfred" valid="yes" autocomplete=""&gt
  &lttitle&gtalfred&lt/title&gt
  &ltsubtitle&gtSearch Google for alfred&lt/subtitle&gt
  &lticon&gticon.png&lt/icon&gt
 &lt/item&gt

 &ltitem uid="2.1613809030" arg="alfred camera" valid="yes" autocomplete=""&gt
  &lttitle&gtalfred camera&lt/title&gt
  &ltsubtitle&gtSearch Google for alfred camera&lt/subtitle&gt
  &lticon&gticon.png&lt/icon&gt
 &lt/item&gt

 &ltitem uid="3.1613809030" arg="alfred &#x4F7F;&#x3044;&#x65B9;" valid="yes" autocomplete=""&gt
  &lttitle&gtalfred &#x4F7F;&#x3044;&#x65B9;&lt/title&gt
  &ltsubtitle&gtSearch Google for alfred &#x4F7F;&#x3044;&#x65B9;&lt/subtitle&gt
  &lticon&gticon.png&lt/icon&gt
 &lt/item&gt

 &ltitem uid="4.1613809030" arg="alfredobannister" valid="yes" autocomplete=""&gt
  &lttitle&gtalfredobannister&lt/title&gt
  &ltsubtitle&gtSearch Google for alfredobannister&lt/subtitle&gt
  &lticon&gticon.png&lt/icon&gt
 &lt/item&gt

 &ltitem uid="5.1613809030" arg="alfred mac" valid="yes" autocomplete=""&gt
  &lttitle&gtalfred mac&lt/title&gt
  &ltsubtitle&gtSearch Google for alfred mac&lt/subtitle&gt
  &lticon&gticon.png&lt/icon&gt
 &lt/item&gt

 &ltitem uid="6.1613809030" arg="alfred camera pc" valid="yes" autocomplete=""&gt
  &lttitle&gtalfred camera pc&lt/title&gt
  &ltsubtitle&gtSearch Google for alfred camera pc&lt/subtitle&gt
  &lticon&gticon.png&lt/icon&gt
 &lt/item&gt

 &ltitem uid="7.1613809030" arg="alfred windows" valid="yes" autocomplete=""&gt
  &lttitle&gtalfred windows&lt/title&gt
  &ltsubtitle&gtSearch Google for alfred windows&lt/subtitle&gt 
  &lticon&gticon.png&lt/icon&gt
 &lt/item&gt

 &ltitem uid="8.1613809030" arg="alfred dunner" valid="yes" autocomplete=""&gt
  &lttitle&gtalfred dunner&lt/title&gt
  &ltsubtitle&gtSearch Google for alfred dunner&lt/subtitle&gt
  &lticon&gticon.png&lt/icon&gt
 &lt/item&gt

 &ltitem uid="9.1613809030" arg="alfred &#x610F;&#x5473;" valid="yes" autocomplete=""&gt
  &lttitle&gtalfred &#x610F;&#x5473;&lt/title&gt
  &ltsubtitle&gtSearch Google for alfred &#x610F;&#x5473;&lt/subtitle&gt
  &lticon&gticon.png&lt/icon&gt
 &lt/item&gt

 &ltitem uid="10.1613809030" arg="alfredo bannister in" valid="yes" autocomplete=""&gt
  &lttitle&gtalfredo bannister in&lt/title&gt
  &ltsubtitle&gtSearch Google for alfredo bannister in&lt/subtitle&gt
  &lticon&gticon.png&lt/icon&gt
 &lt/item&gt

&lt/items&gt
```
<br>確かに<titem>をみると以下のロジックでセットしていることが見て取れますね
	

```
result( $int.'.'.time(), "$data", "$data", 'Search Google for '.$data, 'icon.png'  )
```
<br>とすると、Alfred用のXMLをechoすれば、出力インターフェースが表示されるのかな
<br>なんとなく理解できました

<br>
[トップページに戻る](https://kitanotamotsu.github.io/)


