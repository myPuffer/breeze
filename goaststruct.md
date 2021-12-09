
### 需要的struct信息
```golang
type StructDecl struct {
	Name   string
	Fields []*StructFieldDecl
}

type StructFieldDecl struct {
	Name    string
	Type    string
	JsonTag string
}
```

### 解析struct定义
```golang
func parseLogStruct(file *ast.File) []*StructDecl {

	var structs []*StructDecl
	
	for _, decl := range file.Decls {
		genDecl, ok := decl.(*ast.GenDecl)
		if !ok || genDecl.Tok != token.TYPE {
			continue
		}

		for _, spec := range genDecl.Specs {
			typeSpec, ok := spec.(*ast.TypeSpec)
			if !ok {
				continue
			}

			structType, ok := typeSpec.Type.(*ast.StructType)
			if !ok {
				continue
			}

			structs = append(structs, &StructDecl{
				Name:   typeSpec.Name.Name,
				Fields: extractStructFields(structType.Fields),
			})
		}
	}
	return structs
}

func loadPackage() *packages.Package {
	cfg := &packages.Config{
		Mode:  packages.NeedFiles | packages.NeedSyntax | packages.NeedTypes | packages.NeedName,
		Tests: false,
	}
	pkgs, err := packages.Load(cfg, "operates.go")
	if err != nil {
		log.Fatal(err)
	}
	if len(pkgs) != 1 {
		log.Fatalf("error: %d packages found", len(pkgs))
	}
	return pkgs[0]
}

func extractStructFields(fl *ast.FieldList) []*StructFieldDecl {
	var fields []*StructFieldDecl
	for _, field := range fl.List {
		typ, ok := field.Type.(*ast.Ident)
		if !ok {
			continue
		}
		typeName := typ.Name

		for _, name := range field.Names {
			fields = append(fields, &StructFieldDecl{
				Name:    name.Name,
				Type:    typeName,
				JsonTag: extractJsonTag(field.Tag.Value),
			})
		}
	}
	return fields
}
```

### 解析tag
```golang
func extractJsonTag(meta string) string {
	re := regexp.MustCompile(`json:"(\w+)"`)
	matches := re.FindAllStringSubmatch(meta, -1)
	if len(matches) > 0 {
		return matches[0][1]
	}
	return ""
}
```
