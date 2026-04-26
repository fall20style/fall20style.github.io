Neovim에서 TypeScript 프로젝트 탐색 설정

## Neovim + TypeScript 최적화 설정 가이드## 1. 파일 경로

* 설정 파일: `~/.config/nvim/init.vim`
* 이 파일을 수정 후 Neovim을 재시작하거나 :source $MYVIMRC를 입력해야 적용됨.

## 2. 필수 플러그인 설치
vim-plug를 사용하여 아래 내용을 추가함.

```
call plug#begin()
  " CoC 본체 (TypeScript 언어 서버 통신 전용)  Plug 'neoclide/coc.nvim', {'branch': 'release'}

  " 구문 하이라이팅 강화  Plug 'sheerun/vim-polyglot'call plug#end()
```

적용: `:PlugInstall 실행 후 :CocInstall coc-tsserver coc-eslint coc-prettier` 입력함.

## 3. 핵심 단축키 (코드 탐색용)
심볼 이동과 가독성을 위한 필수 매핑임. 파일 하단에 추가함.

```
{
" K: 타입 정보 및 문서 팝업 표시 (Hover)nnoremap <silent> K :call ShowDocumentation()<CR>
function! ShowDocumentation()
  if CocAction('hasProvider', 'hover')
    call CocActionAsync('doHover')
  else
    call feedkeys('K', 'in')
  endifendfunction
" gd: 정의로 이동 (Go Definition) - 다른 파일로 점프함
nmap <silent> gd <Plug>(coc-definition)
" gy: 타입 정의로 이동 (Go Type Definition)
nmap <silent> gy <Plug>(coc-type-definition)
" gi: 구현체로 이동 (Go Implementation)
nmap <silent> gi <Plug>(coc-implementation)
" gr: 참조된 곳 찾기 (Go References)
nmap <silent> gr <Plug>(coc-references)
" Ctrl + o: 이전 위치로 돌아오기 (Jump Back)
" Ctrl + i: 다시 앞으로 가기 (Jump Forward)
{
```

## 4. TypeScript 가독성 향상 팁
코드를 더 잘 읽기 위해 :CocConfig를 입력하고 아래 내용을 추가함.

```
{
  "coc.preferences.formatOnSaveFiletypes": ["typescript", "typescriptreact"],
  "tsserver.implicitProjectConfig.checkJs": true,
  "diagnostic.displayByAle": false,
  "suggest.noselect": true
}
```
{

## 프로젝트 초기화 체크리스트

1. 의존성 설치: 프로젝트 루트에서 npm install 실행함.
2. 설정 파일: 루트에 tsconfig.json이 있는지 확인함.
3. 서버 재시작: 설정이 꼬이면 언제든 :CocRestart 입력함.

이제 gd로 자유롭게 파일을 넘나들며 코드를 읽을 수 있음.

