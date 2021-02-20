
## Google suggest ワークフローを解析してみる

phpはよく知らないのですが、気になってGoogle suggest ワークフローを動かしながら解析してみました

<pre>
require_once('workflows.php');
$wf = new Workflows();
</pre>

ソースのはじめは、$wfをWorkflowsオブジェクトとしてクリエイトしている感じですね。
workflows.phpには、workflowsオブジェクトのクラスが記述されているのでしょう。
雰囲気を掴みたいだけなのでスルーします。

<pre>
$orig = $argv[1];
$xml = $wf->request( "https://suggestqueries.google.com/complete/search?output=toolbar&q=".urlencode( $orig ) );
$xml = simplexml_load_string( utf8_encode($xml) );
</pre>

アルフの受け渡し（argv[1]）からXMLを取得しているようですね。

https://suggestqueries.google.com/complete/search?output=toolbar&q=
このアドレスでgoogle suggestのxmlが取得できるのでしょう。
ターミナルでopenしてみたら、思った通り下記のxmlが帰ってきました
ちょっと長いですが全部コピペします
因みにq=alfredとして検索しています

<pre>
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
</Pre>

構造は簡単ですね。dataの中にキーワードと関連する検索ワードが10個列挙されるようです。
なお、suggestqueries.google.comをgoogle.comに変更しても、現状では同じxmlが帰ってきます。

<pre>
$int = 1;
if ($xml) {
	foreach( $xml as $sugg ):
		$data = $sugg->suggestion->attributes()->data;
		$wf->result( $int.'.'.time(), "$data", "$data", 'Search Google for '.$data, 'icon.png'  );
		$int++;
	endforeach;
}
</pre>

リターンされたXMLのsuggestionタグのdata属性の内容を見て、リターンを作成しているようです。

<pre>
$results = $wf->results();
if ( count( $results ) == 0 ):
	$wf->result( 'googlesuggest', $orig, 'No Suggestions', 'No search suggestions found. Search Google for '.$orig, 'icon.png' );
endif;
</pre>

こちらはリターンが何もない時のロジックのようです。まあ無視しましょう。
<pre>
echo $wf->toxml();
</pre>

上記で作成したリターンからAlfred用のXMLを整形しているようです。
デバッグしてみたらAlfred用のXMLをみることができました。
コピペします。
なお、XMLは1文字入力するごとに作成されます。というか、1文字入力するごとにgoogle suggestが起動されるようです。まぁAlfredがインクリメンタルサーチなので当然か。

<pre>
<?xml version="1.0"?>
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
</pre>

確かにitemをみると以下のロジックでセットしているのですね

result( $int.'.'.time(), "$data", "$data", 'Search Google for '.$data, 'icon.png'  )

とすると、Alfred用のXMLをechoすれば、出力インターフェースが表示されるのでしょう
なんとなく理解できました


