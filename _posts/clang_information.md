clang information:



A `tooling::ClangTool` runs on an action. A `std::unique_pointer<FrontendActionFactory>` (a wrapper of an action) can be created from `tooling::newFrontendActionFactory<SomeAction>()`. 

The action is usually inherited from `clang::ASTFrontendAction`, with the function `CreateAstConsumer` overridden to return a polymorphic `clang::ASTConsumer` which we want to override `HandleTranslationUnit`. According to the docs:

> HandleTranslationUnit - This method is called when the ASTs for entire translation unit have been parsed. 

The consumer would want to use a custom AST visitor to traverse the populated nodes.

`SourceManager` is an object that takes care of a single source file. You can get `SourceLocation`s from SourceManager which can then be "promoted" to `FullSourceLoc`s which has the exact location of the AST node in the source.

You can also get `FileEntry` associated with the `SourceManager`. 

<script src="https://utteranc.es/client.js" repo="OneRaynyDay/oneraynyday.github.io" issue-term="pathname" theme="github-light" crossorigin="anonymous" async> </script>
