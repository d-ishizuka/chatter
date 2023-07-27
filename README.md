# chatter

# API設定
## 7
- frontディレクトリの作成
 - npx create-next-app client --ts
- backendディレクトリの作成(api)
 - npm init -y
 - package.jsonの変更
 - server.jsの作成

## 8
- experssとのnodemonをインストール
- server.jsでexpressサーバーを立てる
 - app.listen
 - npm run dev(package.jsonに設定済)
 - getのエンドポイントを試しに作ってみる

## 9
- データベースの用意
 - supabaseにログイン
  - 簡単に設定できるので使う(backend as a service)
 - new projectを作成
  - DB用のパスワード設定する
  - postgresにする

## 10
- prisma
 - ORMとして機能する
 - サイトに行って、create project setupに従って設定
 - npx prisma initするとディレクリが作成されてschemaファイルができる
 - .envにsupabaseに書いたパスワードなどを設定する

## 11
- モデルを作成する
 - UserモデルとPostモデルを作成する
  - autoincrementでid自動で振れる
  - @uniqueでユニークキー
  - []で配列指定 
  - @relation
 - migrationする
  - sql文に生成する
  - supabaseに繋がってないとだめ

## 12
- ユーザー登録のAPI作成
    - app.post(path, ...)
    - req.bodyで受け取る
    - prisma clientインストールしてimportする
    - create, findManyとかprismaの作法がある(ORMなので)
    - schema prismaに設定されているモデルの小文字が呼び出せる(User→user)
        - ActiveRecordのようにルールがある

## 13
- パスワードの暗号化
    - bcryptをインストールしてインポート
    - bcryptでハッシュ化
        - 使い方はドキュメント参照

## 14
- Thunder Clientを使うとpostman的なことができる
- express側でjson形式であることを設定する
- supabaseにデータを保存されている

## 15
- ユーザーログインのAPIを作成
    - Tokenを発行するAPI(/api/auth/login)
    - email, passwordを受け取ってuserを検索
    - ユーザー、パスワード見つからなかったらエラーを返す

## 16
- JWT(JSON WEB TOKEN)を発行して返す
    - jsonwebtokenをインストールしてimport
    - user idをもとに、jwtを発行
        - expireの起源とかも決める
        - SECRET_KEYはかなり長いものを指定するのが一般的
        - .envから呼び出してgitに上げない
    - SECRET_KEYを.envから呼び出すために、dotenvをインストールしてrequire

## 17
- Thunder Clientで投げてみる
- jwtのサイトいくと、発行されたjwtをデコードできる

## 18
- apiファイルを分割する
 - routersディレクトリを作成してserver.jsに書いてた各種エンドポイントを分割する
  - auth.js, user.js...
  - 各種ファイルでrouterをrequireで呼び出して、module.exports = routerでエクスポート
  - server.js側で各routerを呼び出す(app.useでルートと、authRouteとかを結びつける)
  - Thunder Clientで正しいか確認する

# Front設定
## 19
- tailwindのセットアップをする
    - next.js tailwindcdでググる
    - npm install -D tailwindcss postcss autoprefixer
    - tailwind.config.jsを修正する
    - global.cssを修正する
    - index.tssでなんか適当にスタイル当てて効いてるか確認

## 20
- sns用のナビゲーションバーを作成する
    - componetnsの中に、Navbar.tsxを作成する
        - これはどのページからも呼び出されるコンポーネント
        - _app.tsxで読んでおけば、どこのページでも呼び出せる
            - application.html.erbと同じようなノリ

## 21
- 投稿用のタイムラインコンポーネントを作成する
    - components/Timeline.tsx
    - index.tsにインポートする
    - Post.tsxを一旦ハードコーディングで作成する
    - Timeline.tsxのcomponentsの中で呼び出すことで、擬似的なタイムラインが表示できる
    - この後、Post.tsxをループでDBからとって表示できるようにする

## 22
- 新規登録フォームとログインフォームを作成する
    - loginページを作りたいので、api/login.tsxを作る
    - global.cssをいじると、サイトの背景色とかも一括で変更できる
    - signupページを作りたいので、api/signup.tsxを作る

## 23, 24
- フォームに入力した値を取得する
    - useStateで入力された値を取ってくる
        - e.g. const [name, setName] = useState<string>("");
        - e.g. フォームのinputタグの中で、onChange={(e) => setName(e.target.value)}
    - formタグのところにonSubmit={() => handleSubmit}を設定する
        - const handleSubmit = (e: any) => {e.preventDefault(); console.log(name)}とかすると、submitのイベントを中断して、送信しようとしている値を確認することができる
        - preventして、この値をapiにとくれば良い
            - 送信するために、axiosをインポートする(fetch関数でもいい)
            - lib/apiClient.tsファイルを作成する
                - axiosをimportして、axios.createする
                    - baseUrlとか設定する
                    - headerでcontentTypeとかも設定する
            - signup.tsxでこの関数を呼んで実行する
                - pathはbaseUrlの以降を指定
                - apiの時に、server.jsでroute設定しているのでそれに合わせればいい
        - 成功したら、リダイレクトする
            - route.push(リダイレクト先)で設定できる
            - 具体的には、ユーザー登録したらログインページに遷移させる
## 25
- corsのエラーを解決する
    - localhost:3000 → localhost:5000へのアクセスを許可する
    - apiがのserver.js側でcorsをインストールして、requireしてuseすることでcorsの許可ができる
        - 実際にはちゃんとアクセス元を絞る必要がある
    - 今回で言うと、auth.jsでリクエストを処理するが、postリクエストする時のattributesは送信側と受け取る側で一致してないとダメなので注意

## 26
- Next.js側からログインAPIを叩く
    - 基本、signup.tsxを参考に修正
    - login.tsxから登録済みのユーザーネーム・emailを送信するとjwtがresponseとして返ってくる
        - response.data.tokenでアクセスできる
    - クライアント側は、response.data.tokenの中身をローカルストレージに保存する
        - この処理はどこの処理でも使う可能性があるので、context/auth.tsxで定義しておく

## 27, 28
- 受け取ったjwtトークンをローカルストレージに保存する
 - context/auth.tsxを定義する
    - React.createContextでフックイベントを作ることができ、その関数の中に処理を書く
    - useAuthみたいな関数を定義して、作成したフックイベントをuseContext(AuthContext)みたいに渡すとフックしてこの処理を外から呼べる
    - Proveider関数を定義する
        - 引数はchiledren(詳細は調べる)
        - login関数を中に定義する
            - tokenを受け取っとたら、localStrage.setItem("auth_token", token)みたいにする
        - logout関数を中に定義する
            - localStorage.removeItem("auth_token")
        - const valueの中に、二つの関数をハッシュで入れる
        - returnで<AuthContext.Provider>で囲む
            - {children}の子コンポーネントで、常にvalueが使えるようになる
        - 要するにアプリケーション全体で使いたいものについては、Reactのコンテキストを使うと使えるようになるよという話
            - ログアウトとかどこからでもできるようなイメージ
            - そこまで複雑ではないので、reduxとか使わずuseContextを使った
    - 型を登録する
        - chiledren: ReactNode
        - 関数は、(token: stirng) => voidとか

## 29
- アプリケーション全体でAuthProviderを使う
    - _app.tsxでNavbarとか全体を、<AuthProvider>で囲むこと
        - この時、AuthProveider関数で定義したchildrenは囲まれたNavbarとかpagePropsのcomponent
        - つまり、囲ったこのcomponentの中では、AuthProviderが使えるということ(valueが使える)
    - よく使うものをいちいち親コンポーネントからバケツリレーすると大変なことになるので、こういう書き方する
    - 例えば、login.tsx関数で、useAuth()の返り値(userContextの生成物)を受けることができる

# つぶやき投稿のAPIと最新のつぶやきを取得するAPIを作る
## 30
- api/routers/posts.jsをauth.jsをベースに作成する
    - つぶやきの内容(content)をrequestから受け取る→なければエラー
    - await prisma.post.create()でプリズマのORMを使う
    - res.status(xxx).json(yyy)でresponse設定できる

## 31
- フロント側から叩けるようにする
    - Timeline.tsxから叩けるようにする
    - handleSubmitでイベントを受け取る関数を作って、formのところにonSubmitで設定する
    - handleSubmitでpreventEventして処理したいことを書く→/posts/postにリクエストを投げる
    - server.jsに/posts/postを設定する→postsRouteという名前で、require("./routers/posts")
        - posts.jsにrouter.post("/post", ...)で設定しているところにリクエストがいく
        - xxx.tsxとyyy.jsの役割の違いを理解する
    - 送信内容(content)はuseStateでwatchしておく
    - 送信後contentの中身を空にする→<textarea>にvalueを設定する必要がある

## 32, 33
- 投稿(post)したつぶやきを配列に格納して、一覧で表示する準備をする
- Timeline.tsxにuseStateでlatestPostsを変数として設定する
- client/src/types.tsというファイルを作成する
    - export interface PostTypeでPostの型を定義する。すると、UserTypeのinterfaceでposts: Post[]が定義できる(type.tsファイル)
    - interfaceの名前と、他のファイルで設定されたPostコンポーネントに対して、同一ファイルにimportされたpostという変数名が被ると怒られるのでPostTypeにする
    - 新しく投稿すると、その新しい投稿をprevPostsに追加するような関数を作成しておく(setLatestPosts)
        - latestPosts.mapでlatestPostsの中身を一つずつ展開して表示する
        - keyを設定する必要がある
    - Post.tsxのコンポーネントにpropsを渡すようにする
        - 考え方としては、このコンポーネントはユーザーの値が動的に入るべきコンポーネントなので、propsでユーザー情報を受け取る
            - railsでコンポーネント化するときも、パラメータ渡してたのに近い
        - propsはPostが入っていて、型はPostType
        - 現時点ではリロード時に投稿を取ってくる処理がないので、リロードすると空になる→次修正

## 34
- 最新のつぶやきを10件取得するAPIを作成する(一旦、post.jsに書く)
- 取得の場合は、router.getになる
- パスを/get_latest_postにする
- await prisma.post.findMany({ take: 10, orderBy: { createdAt: "desc"}})みたいな感じ

## 35
- タイムライン読み込み時に最新投稿取得のAPIを叩いてみる
- useEffect(() => const fetchLatestPosts = async () ... 
    - await apiClient.get("/posts/get_latest_post"))で最新データを取得
    - setLatestPostsでlatestPostsを更新する→配列に値が入って表示できるようになる
    - 日時が読みづらいので、コンポーネントのところで、new DateでtoLocalString()に変換
## 36
- つぶやいた人の名前を取得できるようにする
    - postのモデルの中には、userのidは入っているがnameは入っていない
    - postとuserはuser_idが外部キーになるようにリレーションを貼ってあるので、prisma.post.findManyするところで、userをincludeしてあげれば良い
        - include: {user: true}で取れる
    - これだけだと、既存のpostをとる時にincludeされるだけなので、newPostを作成する時にもincludeしてあげる

## 37
- ローカルストレージにトークンがある人だけ認証許可のヘッダーをつけて、ポストとかできるようにする
- まずは、headerにクライアント側が入れて送れるようにする
- client/src/contest/auth.tsxでuseEffectを使うことで、読み込み時に1回だけ発火する
- apiClient.defaults.headers["Authorizaiton"]=　`Bearer ${token}`...にすると、headerの中に入る

## 38
- バックエンド側で、authorizationの値を確認できるようにするため、それ用のミドルウェアを作成する
- ミドルウェアを作成したらpostなどの処理に挟んで認可の確認をする
- isAuthenticated.jsを作成する
    - const token = req.headers.authorizaiton?.split(" ")[1] ← Bearerの文字を消す
    - jwt.verifyすると、jwtがデコードできるので、デコードしたときに、jwt用のsecret_keyと一致すれば?ok(何でもってokにしてるんだろ)
    - okなら、req.userId = decoded.idにして、postの時にuserIdを教えてあげる
    - next()を使うと、これが通ったらというmiddlewareにできる

## 39
- supabaseをいちいちログインしなくても、npx prisma studioを実行するとローカルでも確認できる

## 41
- ログインユーザーを切り替えてもトークンが切り替わらない問題について
    - ログインだけでなくログアウト機能を追加していないのでクッキーのtokenが削除されなかったり・切り替わらない

## 42
- ログインユーザーを取得するためのAPIを作成する
    - posts.jsを参考にuser.jsを作成する
    - router.get("/find", isAuthenticated, ...)
        - isAuthenticatedのミドルウェアから取得したuser_idを使う
        - passwordを返すと脆弱性になるので意図的に返さない

## 43
- Next.js側でリロードするたびにログインユーザーの取得APIを叩く
- useEffectのタイミングつまりページがリロードされたタイミングでAPIを叩く
    - auth.tsxのAuthProvider
    - tokenがローカルストレージに存在した場合に処理が走るように変更
        - 取得できたら、user変数に代入（useStateを使って実装[user, setUser]）
        - userをアプリケーション全体で使用するためにvalueの中に入れておく
        - loginのタイミングでも取得する
    - server.js側で、usersRouteをrequireしつつ、app.useでパスとusersRouteを紐づける

## 44
- nabvarでuser情報を取得できているか確認
- Navbar.tsxでuseAuth()を呼び出す
- userが存在していたらプロフィールボタンにリンクを貼って遷移できるようにする
- ついでにログアウトボタンも実装する

## 45
- ログアウト機能の実装
- 1.localStorageからauth_tokenをremoveする
- 2.user変数の状態を変更する(setUserを使う)
- 3.headerからAuthorizationを削除する

## 46
- ログアウトしてログインしたのにうまくいかない理由を確認する
- isAuthenticatedとかで正しいuser_idが取れてないのか？？
- 再度ログインしても前回のユーザーが残っちゃう
- 原因は、loginの処理の中で、headerにAuthorizationを設定し直すのを忘れていた

## 4７
- プロフィール用のモデルを作る
- schema.prismaに追加する
    - id : 主キー
    - bio, imageUrl
    - userId: リレーション設定する
    - Userモデル側にもprofileを設定
- npx prisma migrate devでマイグレーション

## 48
- auth.jsでユーザー作成しているところがあるのでprofileのattributeのところでcreateする
- supabaseは１週間ログインしないと動かなくなるので注意

## 49
- identiconを使ってユーザーのアイコンを自動で生成する
    - npm install identicon.js --save
- api/utils/generateIdenticon.jsでidenticon用の関数を作成
    - cryptoを使ってinputデータに対してhashを作成
- module.exportsしてどこからでも参照できるようにする

## 50
- アイコンを生成する
    - auth.jsのユーザー作成のタイミングでemailを渡してhash化及び画像生成する
- api側でprofileをincludeすることでフロントエンド側でuser情報の取得に合わせてprofile情報も取得でき、アイコンのurlが取得できる
- front側でもUserとProfileの型を変更する

## 51
- 画像のパスが通っていない問題を解決する
- includeの仕方が間違っていた
    - 正しくは、include → author → include → profile: true

## 52
- プロフィール専用ページを作成する
- profile/{userId}
- Next.jsの動的ルーティング
    - pages/profile/[userId].tsx
    - 上記のようなファイルを作るとパスが飛べるようになる

## 53
- 閲覧ユーザーのプロフィール情報を取得するAPIを作成する
- user.jsファイル
    - getでパスに "/profile/:userId"を設定すると、profile/1に飛んだ時に、userIdがreq.paramsとかで取得できるようになる
    - 