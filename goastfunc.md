
### 需要的func信息

```golang
type FuncField struct {
	FieldName string
	FieldType string
}

type FuncDecl struct {
	FuncName    string
	In          []*FuncField
	Out         []*FuncField
}
```

### 解析func定义

```golang
func parseFuncDecls(f *ast.File) []*FuncDecl {

	var funcs []*FuncDecl
	for _, decl := range f.Decls {
		if funcDecl, ok := decl.(*ast.FuncDecl); ok {
			if funcDecl.Doc == nil {
				continue
			}
			var toGenerate = false
			for _, comment := range funcDecl.Doc.List {
				if comment.Text == fmt.Sprintf("//%s +async", funcDecl.Name.String()) {
					toGenerate = true
				}
			}
			if !toGenerate {
				continue
			}

			fd := &FuncDecl{
				FuncName: funcDecl.Name.String(),
			}
			for _, field := range funcDecl.Type.Params.List {
				fieldType := field.Type.(*ast.Ident).Name
				for _, name := range field.Names {
					fd.In = append(fd.In, &FuncField{
						FieldName: name.String(),
						FieldType: fieldType,
					})
				}
			}
			if funcDecl.Type.Results != nil {

				for _, field := range funcDecl.Type.Results.List {
					fieldType := field.Type.(*ast.Ident).Name
					for _, name := range field.Names {
						fd.Out = append(fd.Out, &FuncField{
							FieldName: name.String(),
							FieldType: fieldType,
						})
					}
					if field.Names == nil {
						fd.Out = append(fd.Out, &FuncField{
							FieldType: fieldType,
						})
					}
				}
			}		
			funcs = append(funcs, fd)
		}
	}
	return funcs
}
```
