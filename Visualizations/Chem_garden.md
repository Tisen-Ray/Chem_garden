

```dataviewjs
// ===== 配置区域 =====
const FOLDERS_TO_INCLUDE = [" "]  // 要搜索的文件夹列表
const FOLDERS_TO_EXCLUDE = ["attachments", "Templates"]  // 要排除的文件夹
const MIN_PARAGRAPH_LENGTH = 10  // 最小段落长度(字符数)
const MAX_RESULTS = 3  // 最大显示结果数
const CLOZE_PERCENTAGE = 0.3  // 挖空比例(0-1)
const CLOZE_SYMBOL = "_____"  // 挖空占位符

// ===== 主逻辑 =====
function getRandomElements(array, count) {
    const shuffled = [...array].sort(() => 0.5 - Math.random())
    return shuffled.slice(0, Math.min(count, array.length))
}

function createCloze(paragraph) {
    const words = paragraph.split(/\s+/)
    const wordsToHide = Math.floor(words.length * CLOZE_PERCENTAGE)
    
    // 随机选择要挖空的单词索引
    const hiddenIndices = new Set()
    while (hiddenIndices.size < wordsToHide) {
        const randomIndex = Math.floor(Math.random() * words.length)
        if (words[randomIndex].length > 3) {  // 只挖空长度大于3的单词
            hiddenIndices.add(randomIndex)
        }
    }
    
    // 创建挖空段落
    return words.map((word, index) => 
        hiddenIndices.has(index) ? CLOZE_SYMBOL : word
    ).join(' ')
}

// ===== 执行流程 =====
try {
    // 获取所有符合条件的Markdown文件
    const files = app.vault.getMarkdownFiles()
        .filter(file => {
            const inIncludedFolder = FOLDERS_TO_INCLUDE.some(folder => 
                file.path.startsWith(folder + "/")
            )
            const inExcludedFolder = FOLDERS_TO_EXCLUDE.some(folder => 
                file.path.startsWith(folder + "/")
            )
            return inIncludedFolder && !inExcludedFolder
        })
    
    // 收集所有符合条件的段落
    let allParagraphs = []
    
    for (const file of files) {
        const content = await app.vault.cachedRead(file)
        const paragraphs = content.split('\n\n')  // 按空行分割段落
        
        paragraphs.forEach(para => {
            const cleanPara = para.replace(/^#+\s+/gm, '').trim()  // 移除标题标记
            if (cleanPara.length >= MIN_PARAGRAPH_LENGTH && 
                !cleanPara.startsWith('```') &&  // 排除代码块
                !cleanPara.startsWith('|')) {    // 排除表格
                allParagraphs.push({
                    text: cleanPara,
                    source: file.path
                })
            }
        })
    }
    
    // 显示结果
    if (allParagraphs.length === 0) {
        dv.paragraph("没有找到符合条件的段落")
    } else {
        const selected = getRandomElements(allParagraphs, MAX_RESULTS)
        
        selected.forEach((item, index) => {
            dv.paragraph(`### 复习卡片 ${index + 1}`)
            dv.paragraph(createCloze(item.text))
            dv.paragraph(`**来源:** ${dv.fileLink(item.source)}`)
            dv.paragraph("---")
        })
        
        dv.paragraph(`*共找到 ${allParagraphs.length} 个段落，随机显示 ${selected.length} 个*`)
    }
} catch (error) {
    dv.paragraph(`❌ 发生错误: ${error.message}`)
}
```