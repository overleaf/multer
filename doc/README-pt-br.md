# Multer [![NPM Version][npm-version-image]][npm-url] [![NPM Downloads][npm-downloads-image]][npm-url] [![Build Status][ci-image]][ci-url] [![Test Coverage][test-image]][test-url] [![OpenSSF Scorecard Badge][ossf-scorecard-badge]][ossf-scorecard-visualizer]

Multer é um middleware node.js para manipulação `multipart/form-data`, que é usado principalmente para fazer upload de arquivos. Foi escrito em cima do [busboy](https://github.com/mscdex/busboy) para máxima eficiência.

**NOTA**: Multer não processará nenhum formulário que não seja multipart (`multipart/form-data`).

## Traduções

Este README também está disponível em outros idiomas:

- [English](https://github.com/expressjs/multer/blob/main/README.md) (Inglês)
- [العربية](https://github.com/expressjs/multer/blob/main/doc/README-ar.md) (Árabe)
- [Español](https://github.com/expressjs/multer/blob/main/doc/README-es.md) (Espanhol)
- [简体中文](https://github.com/expressjs/multer/blob/main/doc/README-zh-cn.md) (Chinês)
- [한국어](https://github.com/expressjs/multer/blob/main/doc/README-ko.md) (Coreano)
- [Русский язык](https://github.com/expressjs/multer/blob/main/doc/README-ru.md) (Russo)
- [Việt Nam](https://github.com/expressjs/multer/blob/main/doc/README-vi.md) (Vietnã)
- [Português](https://github.com/expressjs/multer/blob/main/doc/README-pt-br.md) (Português Brasil)
- [Français](https://github.com/expressjs/multer/blob/main/doc/README-fr.md) (Francês)
- [O'zbek tili](https://github.com/expressjs/multer/blob/main/doc/README-uz.md) (Uzbequistão)

## Instalação

```sh
$ npm install --save multer
```

## Uso

Multer adiciona um objeto `body` e um `file` ou objeto `files` para objeto `request`. O objeto `body` contém os valores dos campos de texto do formulário, o objeto `file` ou `files` contém os arquivos enviados por meio do formulário.

Exemplo de uso básico:

Não esqueça o `enctype="multipart/form-data"` em seu formulário.

```html
<form action="/profile" method="post" enctype="multipart/form-data">
  <input type="file" name="avatar" />
</form>
```

```javascript
const express = require('express')
const multer = require('multer')
const upload = multer({ dest: 'uploads/' })

const app = express()

app.post('/profile', upload.single('avatar'), function (req, res, next) {
  // req.file é um arquivo `avatar`
  // req.body conterá os campos de texto, se houver
})

app.post('/photos/upload', upload.array('photos', 12), function (req, res, next) {
  // req.files é um array de arquivos `photos`
  // req.body conterá os campos de texto, se houver
})

const uploadMiddleware = upload.fields([{ name: 'avatar', maxCount: 1 }, { name: 'gallery', maxCount: 8 }])
app.post('/cool-profile', uploadMiddleware, function (req, res, next) {
  // req.files é um objeto (String -> Array) onde fieldname é a chave e o valor é array de arquivos
  //
  // e.g.
  //  req.files['avatar'][0] -> File
  //  req.files['gallery'] -> Array
  //
  // req.body conterá os campos de texto, se houver
})
```

Caso você precise lidar com formulário text-only multipart, você deve usar o método `.none()`:

```javascript
const express = require('express')
const app = express()
const multer  = require('multer')
const upload = multer()

app.post('/profile', upload.none(), function (req, res, next) {
  // req.body contém os campos de texto
})
```

Aqui está um exemplo de como o multer é usado em um formulário HTML. Onde adicionamos `enctype="multipart/form-data"` no form e no input `name="uploaded_file"`:

```html
<form action="/stats" enctype="multipart/form-data" method="post">
  <div class="form-group">
    <input type="file" class="form-control-file" name="uploaded_file">
    <input type="text" class="form-control" placeholder="Número de palestrantes" name="nspeakers">
    <input type="submit" value="Obter as estatísticas!" class="btn btn-default">
  </div>
</form>
```

Então, em seu arquivo javascript, você adicionaria essas linhas para acessar o arquivo e o corpo. É importante que você use o valor do campo `name` do formulário em sua função de upload. Isso informa ao multer em qual campo da solicitação ele deve procurar os arquivos. Se esses campos não forem iguais no formulário HTML e no seu servidor, seu upload falhará:

```javascript
const multer  = require('multer')
const upload = multer({ dest: './public/data/uploads/' })
app.post('/stats', upload.single('uploaded_file'), function (req, res) {
  // req.fileé o nome do seu arquivo no formato acima, aqui 'uploaded_file'
  // req.body irá conter os campos de texto, se houver algum
  console.log(req.file, req.body)
});
```

## API

### Informação de arquivo

Cada arquivo contém as seguintes informações:

Key | Descrição | Nota
--- | --- | ---
`fieldname` | Nome do campo especificado no formulário |
`originalname` | Nome do arquivo no computador do usuário |
`encoding` | Tipo de codificação do arquivo |
`mimetype` | Tipo Mime do arquivo |
`size` | Tamanho do arquivo em bytes |
`destination` | A pasta na qual o arquivo foi salvo | `DiskStorage`
`filename` | O nome do arquivo dentro do `destination` | `DiskStorage`
`path` | O caminho completo para o arquivo enviado | `DiskStorage`
`buffer` | O `Buffer` do arquivo inteiro | `MemoryStorage`

### `multer(opts)`

Multer aceita um objeto de opções, a propriedade mais básica é o `dest`, que diz ao Multer onde fazer o upload dos arquivos. No caso de você omitir o objeto de opções, os arquivos serão mantidos na memória e nunca gravados no disco.

Por padrão, Multer irá renomear os arquivos para evitar conflitos de nomes. A função de renomeação pode ser personalizada de acordo com suas necessidades.

A seguir estão as opções que podem ser passadas para o Multer.

Key | Descrição
--- | ---
`dest` ou `storage` | Onde armazenar os arquivos
`fileFilter` | Função para controlar quais arquivos são aceitos
`limits` | Limites dos dados enviados
`preservePath` | Mantenha o caminho completo dos arquivos em vez de apenas o nome base

Em um web app básico, somente o `dest` pode ser necessário, e configurado como mostrado no exemplo a seguir:

```javascript
const upload = multer({ dest: 'uploads/' })
```

Se você quiser mais controle sobre seus envios, você ter que usar a opção `storage` em vez de `dest`. Multer vem com motores de armazenamento `DiskStorage` e `MemoryStorage`; Mais mecanismos estão disponíveis de terceiros.

#### `.single(fieldname)`

Aceite um único arquivo com o nome `fieldname`. O arquivo único será armazenado em `req.file`.

#### `.array(fieldname[, maxCount])`

Aceite múltiplos arquivos, todos com o nome `fieldname`. Opcional, gera um errose forem enviados mais de `maxCount`. O array de arquivos serão armazenados em
`req.files`.

#### `.fields(fields)`

Aceita uma mistura de arquivos, especificada por `fields`. Um objeto com um array de arquivos será armazenado em `req.files`.

`fields` deve ser uma matriz de objetos com `name` e opcionalmente com `maxCount`.

Exemplo:

```javascript
[
  { name: 'avatar', maxCount: 1 },
  { name: 'gallery', maxCount: 8 }
]
```

#### `.none()`

Aceite apenas campo de texto. Se algum upload de arquivo for feito, um erro com código "LIMIT\_UNEXPECTED\_FILE" será emitido.

#### `.any()`

Aceita todos os arquivos que são enviados. Uma matriz de arquivos será armazenada em
`req.files`.

**AVISO:** Certifique-se de sempre manipular os arquivos que um usuário envia.
Nunca adicione o Multer como global no middleware, já que um usuário mal-intencionado poderia fazer upload de arquivos para uma rota que você não previu. Use esta função apenas nas rotas onde você está lidando com os arquivos enviados.

### `storage`

#### `DiskStorage`

O mecanismo de armazenamento em disco oferece controle total sobre o armazenamento de arquivos em disco.

```javascript
const storage = multer.diskStorage({
  destination: function (req, file, cb) {
    cb(null, '/tmp/my-uploads')
  },
  filename: function (req, file, cb) {
    cb(null, file.fieldname + '-' + Date.now())
  }
})

const upload = multer({ storage: storage })
```

Existem duas opções disponíveis, `destination` e `filename`. Ambas são funções que determinam onde o arquivo deve ser armazenado.

`destination` é usado para determinar em qual pasta os arquivos enviados devem ser armazenados. Isso também pode ser dado como uma `string` (e.g. `'/tmp/uploads'`). Se não é dada `destination`, o diretório padrão do sistema operacional para arquivos temporários é usado.

**Nota:** Você é responsável por criar o diretório ao fornecer o "destino" com uma função. Ao passar uma string, o Multer se certificará de que o diretório foi criado para você.

`filename` é usado para determinar qual arquivo deve ser nomeado dentro da pasta.
Se não for passado `filename`, cada arquivo receberá um nome aleatório que não inclui nenhuma extensão de arquivo.

**Nota:** Multer não adicionará nenhuma extensão de arquivo para você, sua função é retornar um nome para o arquivo completo com a extensão de arquivo.

Cada função é passada pelo request (`req`) e algumas informações sobre o arquivo (`file`) para ajudar com a decisão.

Observe que `req.body` pode não ter sido totalmente preenchido ainda. Isso depende da ordem na qual o cliente transmite campos e arquivos para o servidor.

Para entender a convenção de chamada usada no callback (precisando passar
null como o primeiro parâmetro), consulte em
[Manipulação de erros no Node.js](https://web.archive.org/web/20220417042018/https://www.joyent.com/node-js/production/design/errors)

#### `MemoryStorage`

O mecanismo de armazenamento na memória, armazena os arquivos na memória como um objeto `Buffer`. Não tendo opções.
```javascript
const storage = multer.memoryStorage()
const upload = multer({ storage: storage })
```
Ao usar o armazenamento de memória, as informações do arquivo conterão um campo chamado `buffer` que contém o arquivo inteiro.

**AVISO**: Fazer upload de arquivos muito grandes ou arquivos relativamente pequenos em grande número muito rapidamente pode fazer com que o aplicativo fique sem memória quando o armazenamento de memória é usado.

### `limits`

Um objeto que especifica os limites de tamanho das seguintes propriedades opcionais. O Multer passa diretamente o objeto para o busboy, e os detalhes das propriedades podem ser encontrados em [busboy's page](https://github.com/mscdex/busboy#busboy-methods).

Os seguintes valores inteiros estão disponíveis:

Key | Descrição | Padrão
--- | --- | ---
`fieldNameSize` | Tamanho máximo do nome de campo| 100 bytes
`fieldSize` | Tamanho máximo do valor do campo (in bytes) | 1MB
`fields` | Max number of non-file fields | Infinity
`fileSize` | Para formulários multipart, o tamanho máximo do arquivo (in bytes) | Infinity
`files` | Para formulários multipart, o número máximo de campos de arquivos | Infinity
`parts` | Para formulários multipart, o número máximo de parts (fields + files) | Infinity
`headerPairs` | Para formulários multipart, o número máximo do header key=>value, para analisar | 2000

A especificação dos limites pode ajudar a proteger seu site contra ataques de negação de serviço (DoS).

### `fileFilter`

Defina isso para uma função para controlar quais arquivos devem ser enviados e quais devem ser ignorados.

A função deve ficar assim:

```javascript
function fileFilter (req, file, cb) {

  // A função deve chamar `cb` com um booleano
  // para indicar se o arquivo deve ser aceito

  // Para rejeitar este arquivo passe `false`, assim:
  cb(null, false)

  // Para aceitar o arquivo passe `true`, assim:
  cb(null, true)

  // Você sempre pode passar um erro se algo der errado:
  cb(new Error('I don\'t have a clue!'))

}
```

## Error handling

Quando encontrar um erro, Multer delegará o erro para Express. Você pode exibir uma boa página de erro usando [the standard express way](http://expressjs.com/guide/error-handling.html).

Se você quer pegar erros especificamente do Multer, você pode enviar para o função de middleware. Além disso, se você quiser pegar apenas [os erros do Multer](https://github.com/expressjs/multer/blob/main/lib/multer-error.js), você pode usar a classe `MulterError` que está ligado ao objeto `multer` (e.g. `err instanceof multer.MulterError`).

```javascript
const multer = require('multer')
const upload = multer().single('avatar')

app.post('/profile', function (req, res) {
  upload(req, res, function (err) {
    if (err instanceof multer.MulterError) {
      // Ocorreu um erro durante o upload.
    } else if (err) {
      // Ocorreu um erro durante o upload.
    }

    // Tudo correu bem.
  })
})
```

## Mecanismo de armazenamento personalizado

Para obter informações sobre como criar seu próprio mecanismo de armazenamento, veja [Multer Storage Engine](https://github.com/expressjs/multer/blob/main/StorageEngine.md).

## Licença

[MIT](LICENSE)

[ci-image]: https://github.com/expressjs/multer/actions/workflows/ci.yml/badge.svg
[ci-url]: https://github.com/expressjs/multer/actions/workflows/ci.yml
[test-url]: https://coveralls.io/r/expressjs/multer?branch=main
[test-image]: https://badgen.net/coveralls/c/github/expressjs/multer/main
[npm-downloads-image]: https://badgen.net/npm/dm/multer
[npm-url]: https://npmjs.org/package/multer
[npm-version-image]: https://badgen.net/npm/v/multer
[ossf-scorecard-badge]: https://api.scorecard.dev/projects/github.com/expressjs/multer/badge
[ossf-scorecard-visualizer]: https://ossf.github.io/scorecard-visualizer/#/projects/github.com/expressjs/multer