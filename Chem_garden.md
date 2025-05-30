

```dataviewjs 
const term = "📌"
const targetFolder = "10-Input"  // ← 修改这里为你的目标文件夹（如："DailyNotes"）
const currentFile = dv.current().file.path

// 获取指定文件夹内的文件
const allFiles = app.vault.getMarkdownFiles()
const files = allFiles.filter(file => 
    file.path.startsWith(targetFolder + "/")  // 匹配文件夹及其子文件夹
)

const arr = files.map(async (file) => {
    const content = await app.vault.cachedRead(file) 
    const lines = content.split("\n")
        .filter(line => line.includes(term))
        .map(line => ({
            file: file,
            content: line
        }));
    return lines 
})

function generateArray(start, end) { 
    return Array.from(new Array(end + 1).keys()).slice(start) 
} 

Promise.all(arr).then(values => {
    // 扁平化数组并排除当前文件
    let noteArr = values.flat().filter(note => note.file.path !== currentFile)
    
    // 当匹配笔记不足时处理
    if (noteArr.length === 0) {
        dv.paragraph("没有找到包含📌的笔记")
        return
    }
    
    let arrNum = generateArray(0, noteArr.length-1)
    let result = []
    let ranNum = Math.min(3, noteArr.length)  // 不超过现有数量
    
    // 无重复随机选择算法
    for (let i = 0; i < ranNum; i++) { 
        let ran = Math.floor(Math.random() * (arrNum.length - i))
        result.push(arrNum[ran])
        arrNum[ran] = arrNum[arrNum.length - i - 1]
    }
    
    // 显示结果
    result.forEach(index => {
        let note = noteArr[index]
        dv.paragraph(
            `${note.content}\n` + 
            `→ 来源：${dv.fileLink(note.file.path)}` +  // 添加文件链接
            "\n\n"  // 添加空行分隔
        )
    })
})
```