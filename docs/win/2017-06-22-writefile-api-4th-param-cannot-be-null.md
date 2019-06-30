# WriteFile API 第四个参数不能为 NULL
2017-06-22

WriteFile API 在 win7 下第四个参数 byteswritten 不能和第五个参数同时为空，否则会有 exception。win10 下没有 exception。