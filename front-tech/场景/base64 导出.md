```js
import { Alert } from '@/components'
import { IntlShape } from 'react-intl'

type ExportType = 'csv' | 'awr' | 'html' | 'txt' | 'pdf'

interface IExportOptions {
  onBeforeExport?: () => void // 导出前的回调
  onAfterExport?: () => void // 导出成功后的回调
  onError?: () => void // 导出失败后的回调
}

interface IExportParams {
  extname: ExportType
  fileName: string
  requestFn: () => Promise<{ code: number; data: string; message: string; success: boolean }>
  intl: IntlShape
  options?: IExportOptions
}

const fileTypeList: ExportType[] = ['html', 'txt', 'pdf']
const extMap = new Map<ExportType, string>([
  ['csv', 'text/csv;charset=utf-8'],
  ['awr', 'text/plain'],
])

fileTypeList.forEach((file) => extMap.set(file, `application/${file}`))

/**
 * 生成下载链接
 * @param extname 文件后缀名
 * @param fileName 文件名称
 * @param blob 文件的blob对象
 */
export const createDownloadLink = (extname: ExportType, fileName: string, blob: Blob) => {
  const url = URL.createObjectURL(blob)

  const link = document.createElement('a')
  link.href = url
  link.download = `${fileName}.${extname}`
  document.body.appendChild(link)
  link.click()

  document.body.removeChild(link)
  URL.revokeObjectURL(url)
}

/**
 * 支持导出文件 - 使用后端API返回的base64字符串
 * @param extname 文件后缀名
 * @param fileName 生成的文件名（不带后缀名）
 * @param requestFn 请求函数，返回Promise，resolve参数为后端返回的base64字符串
 */
export const exportFile = async ({
  extname,
  fileName,
  requestFn,
  intl,
  options,
}: IExportParams) => {
  if (!extMap.has(extname)) {
    throw new Error(`不支持该文件类型: ${extname}`)
  }

  if (!fileName || fileName.trim() === '') {
    throw new Error('文件名参数错误')
  }

  const { onBeforeExport, onAfterExport, onError } = options ?? {}
  try {
    onBeforeExport
      ? onBeforeExport()
      : Alert.alert(`${intl.formatMessage({ id: 'Exporting log' })}`)

    const data = await requestFn()
    if (data.success && data.data) {
      const base64 = window.atob(data.data)
      const bytes = new Uint8Array(base64.length)
      for (let i = 0; i < base64.length; i++) {
        bytes[i] = base64.charCodeAt(i)
      }

      const blob = new Blob([bytes], { type: extMap.get(extname) })

      createDownloadLink(extname, fileName, blob)

      onAfterExport
        ? onAfterExport()
        : Alert.alert(`${intl.formatMessage({ id: 'Export Success' })}`, 'success')
    } else {
      onError ? onError() : Alert.alert(`${intl.formatMessage({ id: 'Export Failed' })}`, 'error')
    }
  } catch (error) {
    console.error('export error:', error)
    onError ? onError() : Alert.alert(`${intl.formatMessage({ id: 'Export Failed' })}`, 'error')
  }
}

```