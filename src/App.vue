<template>
  <main class="min-h-screen bg-background text-foreground">
    <input
      ref="fileInput"
      class="hidden"
      type="file"
      :accept="supportedExts.map((ext) => `.${ext}`).join(',')"
      @change="handleFileChange"
    />

    <section class="mx-auto flex min-h-screen w-full max-w-6xl flex-col px-4 pb-24 pt-5 sm:px-6 lg:px-8 lg:pb-8">
      <header class="flex flex-col gap-4 border-b pb-5 sm:flex-row sm:items-center sm:justify-between">
        <div>
          <Badge variant="outline" class="mb-3">Office Converter</Badge>
          <h1 class="text-2xl font-semibold tracking-tight sm:text-3xl">办公文件转 PDF</h1>
          <p class="mt-2 text-sm text-muted-foreground">单文件上传，服务端实时进度，完成后确认下载。</p>
        </div>
        <div class="hidden items-center gap-2 sm:flex">
          <Button variant="outline" :disabled="!currentFile || converting" @click="removeFile">
            <Trash2 class="h-4 w-4" />
            移除文件
          </Button>
          <Button :loading="converting" :disabled="!canConvert" @click="convertFile">
            <FileText class="h-4 w-4" />
            开始转换
          </Button>
        </div>
      </header>

      <div class="grid flex-1 gap-5 py-5 lg:grid-cols-[360px_1fr] xl:grid-cols-[400px_1fr]">
        <aside class="space-y-4">
          <Card
            class="cursor-pointer border-dashed p-6 transition-colors hover:bg-muted/50"
            @click="openFilePicker"
            @dragover.prevent
            @drop.prevent="handleDrop"
          >
            <div class="flex flex-col items-center text-center">
              <div class="rounded-full border bg-background p-4 shadow-sm">
                <UploadCloud class="h-8 w-8 text-muted-foreground" />
              </div>
              <h2 class="mt-4 text-base font-medium">拖拽或点击上传单个办公文件</h2>
              <p class="mt-2 text-sm text-muted-foreground">支持 Word、PPT、Excel，重新上传会替换当前文件。</p>
            </div>
          </Card>

          <div class="grid grid-cols-3 gap-3">
            <Card class="p-3">
              <span class="text-xs text-muted-foreground">当前文件</span>
              <strong class="mt-1 block text-lg">{{ currentFile ? '1' : '0' }}</strong>
            </Card>
            <Card class="p-3">
              <span class="text-xs text-muted-foreground">状态</span>
              <strong class="mt-1 block text-lg">{{ currentStep }}</strong>
            </Card>
            <Card class="p-3">
              <span class="text-xs text-muted-foreground">大小</span>
              <strong class="mt-1 block text-lg">{{ currentFile ? formatSize(currentFile.size) : '0 KB' }}</strong>
            </Card>
          </div>

          <Card class="hidden p-5 lg:block">
            <div class="space-y-5">
              <div class="flex gap-3">
                <div class="mt-0.5 flex h-7 w-7 items-center justify-center rounded-full bg-primary text-primary-foreground">
                  <UploadCloud class="h-4 w-4" />
                </div>
                <div>
                  <p class="font-medium">上传文件</p>
                  <p class="text-sm text-muted-foreground">选择 doc、docx、ppt、pptx、xls、xlsx</p>
                </div>
              </div>
              <div class="flex gap-3">
                <div class="mt-0.5 flex h-7 w-7 items-center justify-center rounded-full bg-secondary text-secondary-foreground">
                  <Loader2 class="h-4 w-4" />
                </div>
                <div>
                  <p class="font-medium">实时转换</p>
                  <p class="text-sm text-muted-foreground">服务端转换时持续轮询任务进度</p>
                </div>
              </div>
              <div class="flex gap-3">
                <div class="mt-0.5 flex h-7 w-7 items-center justify-center rounded-full bg-secondary text-secondary-foreground">
                  <Download class="h-4 w-4" />
                </div>
                <div>
                  <p class="font-medium">确认下载</p>
                  <p class="text-sm text-muted-foreground">完成后询问是否立即下载 PDF</p>
                </div>
              </div>
            </div>
          </Card>

          <Card class="p-4">
            <div class="flex gap-3">
              <ShieldCheck class="mt-0.5 h-5 w-5 text-muted-foreground" />
              <div>
                <p class="font-medium">已接入服务端</p>
                <p class="mt-1 text-sm leading-6 text-muted-foreground">上传、进度轮询和 PDF 下载都通过 Nest 接口完成。</p>
              </div>
            </div>
          </Card>
        </aside>

        <section class="min-w-0">
          <div class="mb-4">
            <h2 class="text-lg font-semibold tracking-tight">单文件转换</h2>
            <p class="text-sm text-muted-foreground">上传一个文件后开始转换，完成后会弹窗确认下载。</p>
          </div>

          <Card v-if="!currentFile" class="grid min-h-[360px] place-items-center border-dashed p-8 text-center">
            <div>
              <div class="mx-auto flex h-14 w-14 items-center justify-center rounded-full bg-muted">
                <UploadCloud class="h-7 w-7 text-muted-foreground" />
              </div>
              <p class="mt-4 font-medium">还没有选择文件</p>
              <p class="mt-2 text-sm text-muted-foreground">上传 Word、PPT 或 Excel 文件后开始转换。</p>
              <Button class="mt-5" @click="openFilePicker">
                <UploadCloud class="h-4 w-4" />
                选择文件
              </Button>
            </div>
          </Card>

          <Card v-else class="p-5 sm:p-6">
            <div class="flex min-w-0 items-center gap-3">
              <div class="flex h-11 w-11 shrink-0 items-center justify-center rounded-md border bg-muted">
                <component :is="fileIcon(currentFile.ext)" class="h-6 w-6 text-foreground" />
              </div>
              <div class="min-w-0 flex-1">
                <div class="flex flex-wrap items-center gap-2">
                  <h3 class="truncate text-base font-semibold sm:text-lg">{{ currentFile.name }}</h3>
                  <Badge :variant="statusVariant(currentFile.status)">{{ statusText(currentFile.status) }}</Badge>
                </div>
                <p class="mt-1 text-sm text-muted-foreground">
                  {{ currentFile.ext.toUpperCase() }} · {{ formatSize(currentFile.size) }}
                </p>
              </div>
            </div>

            <div class="mt-8">
              <div class="mb-3 flex items-center justify-between gap-3 text-sm text-muted-foreground">
                <span class="truncate">{{ currentFile.serverMessage || '转换进度' }}</span>
                <span>{{ currentFile.progress }}%</span>
              </div>
              <Progress :model-value="currentFile.progress" />
            </div>

            <div class="mt-8 flex flex-col gap-3 sm:flex-row sm:justify-end">
              <Button v-if="currentFile.status === 'done'" size="lg" @click="downloadFile(currentFile)">
                <Download class="h-4 w-4" />
                下载 PDF
              </Button>
              <Button v-else size="lg" :loading="converting" :disabled="!canConvert" @click="convertFile">
                <Check class="h-4 w-4" />
                开始转换
              </Button>
              <Button variant="outline" size="lg" :disabled="converting" @click="removeFile">
                <Trash2 class="h-4 w-4" />
                移除文件
              </Button>
            </div>
          </Card>
        </section>
      </div>

      <nav class="mobile-action-bar">
        <Button variant="outline" :disabled="!currentFile || converting" @click="removeFile">移除</Button>
        <Button v-if="currentFile?.status === 'done'" @click="downloadFile(currentFile)">
          <Download class="h-4 w-4" />
          下载 PDF
        </Button>
        <Button v-else :loading="converting" :disabled="!canConvert" @click="convertFile">
          <Check class="h-4 w-4" />
          开始转换
        </Button>
      </nav>
    </section>

    <div v-if="notice" class="fixed right-4 top-4 z-50 max-w-sm rounded-md border bg-card px-4 py-3 text-sm shadow-lg" :class="noticeType === 'destructive' ? 'border-destructive text-destructive' : ''">
      {{ notice }}
    </div>

    <div v-if="showDownloadDialog && completedFile" class="fixed inset-0 z-50 grid place-items-center bg-black/45 px-4">
      <Card class="w-full max-w-md p-6">
        <div class="flex h-11 w-11 items-center justify-center rounded-full bg-primary text-primary-foreground">
          <Download class="h-5 w-5" />
        </div>
        <h2 class="mt-5 text-lg font-semibold tracking-tight">转换完成，是否立即下载？</h2>
        <p class="mt-2 text-sm leading-6 text-muted-foreground">{{ completedFile.name }} 已转换为 PDF 文件。</p>
        <div class="mt-6 flex flex-col-reverse gap-3 sm:flex-row sm:justify-end">
          <Button variant="outline" @click="showDownloadDialog = false">稍后下载</Button>
          <Button @click="downloadFile(completedFile)">
            <Download class="h-4 w-4" />
            下载 PDF
          </Button>
        </div>
      </Card>
    </div>
  </main>
</template>

<script setup lang="ts">
import { computed, ref } from 'vue'
import {
  Check,
  Download,
  FileSpreadsheet,
  FileText,
  Loader2,
  Presentation,
  ShieldCheck,
  Trash2,
  UploadCloud,
} from 'lucide-vue-next'
import { Badge } from '@/components/ui/badge'
import { Button } from '@/components/ui/button'
import { Card } from '@/components/ui/card'
import { Progress } from '@/components/ui/progress'

type ConvertStatus = 'ready' | 'converting' | 'done' | 'error'
type ServerStatus = 'queued' | 'converting' | 'done' | 'error'
type NoticeType = 'default' | 'destructive'

interface OfficeFile {
  uid: string
  name: string
  size: number
  ext: string
  progress: number
  status: ConvertStatus
  raw: File
  taskId?: string
  serverMessage?: string
  downloadUrl?: string
}

interface ConvertProgress {
  taskId: string
  fileName: string
  progress: number
  status: ServerStatus
  message: string
  downloadUrl?: string
}

const apiBaseUrl = import.meta.env.VITE_API_BASE_URL || '/api'
const supportedExts = ['doc', 'docx', 'ppt', 'pptx', 'xls', 'xlsx']
const maxSize = 50 * 1024 * 1024
const currentFile = ref<OfficeFile | null>(null)
const converting = ref(false)
const fileInput = ref<HTMLInputElement | null>(null)
const showDownloadDialog = ref(false)
const completedFile = ref<OfficeFile | null>(null)
const notice = ref('')
const noticeType = ref<NoticeType>('default')
let fileSeed = 0
let progressTimer: ReturnType<typeof setInterval> | null = null
let noticeTimer: ReturnType<typeof setTimeout> | null = null

const canConvert = computed(
  () =>
    Boolean(currentFile.value) &&
    (currentFile.value?.status === 'ready' || currentFile.value?.status === 'error') &&
    !converting.value,
)

const currentStep = computed(() => {
  if (!currentFile.value) return '等待上传'
  if (currentFile.value.status === 'done') return '等待下载'
  if (currentFile.value.status === 'converting') return '正在转换'
  if (currentFile.value.status === 'error') return '转换失败'
  return '准备转换'
})

function openFilePicker() {
  if (converting.value) {
    showNotice('当前文件正在转换，请完成后再重新上传', 'destructive')
    return
  }

  fileInput.value?.click()
}

function handleFileChange(event: Event) {
  const input = event.target as HTMLInputElement
  const [file] = Array.from(input.files ?? [])

  if (file) {
    setFile(file)
  }

  input.value = ''
}

function handleDrop(event: DragEvent) {
  const [file] = Array.from(event.dataTransfer?.files ?? [])

  if (file) {
    setFile(file)
  }
}

function setFile(file: File) {
  if (converting.value) {
    showNotice('当前文件正在转换，请完成后再重新上传', 'destructive')
    return
  }

  const ext = getExt(file.name)

  if (!supportedExts.includes(ext)) {
    showNotice(`${file.name} 暂不支持，请上传 Word、PPT 或 Excel 文件`, 'destructive')
    return
  }

  if (file.size > maxSize) {
    showNotice(`${file.name} 超过 50MB 限制`, 'destructive')
    return
  }

  currentFile.value = {
    uid: `${Date.now()}-${fileSeed++}`,
    name: file.name,
    size: file.size,
    ext,
    progress: 0,
    status: 'ready',
    raw: file,
  }
  showDownloadDialog.value = false
}

function getExt(fileName: string) {
  return fileName.split('.').pop()?.toLowerCase() ?? ''
}

function formatSize(size: number) {
  if (size < 1024 * 1024) {
    return `${(size / 1024).toFixed(1)} KB`
  }

  return `${(size / 1024 / 1024).toFixed(1)} MB`
}

function fileIcon(ext: string) {
  if (ext.startsWith('doc')) return FileText
  if (ext.startsWith('ppt')) return Presentation
  if (ext.startsWith('xls')) return FileSpreadsheet
  return FileText
}

function statusText(status: ConvertStatus) {
  const textMap: Record<ConvertStatus, string> = {
    ready: '待转换',
    converting: '转换中',
    done: '已完成',
    error: '失败',
  }

  return textMap[status]
}

function statusVariant(status: ConvertStatus) {
  if (status === 'done') return 'success'
  if (status === 'error') return 'destructive'
  if (status === 'converting') return 'warning'
  return 'secondary'
}

function removeFile() {
  if (converting.value) return
  currentFile.value = null
  completedFile.value = null
  showDownloadDialog.value = false
}

async function convertFile() {
  const file = currentFile.value

  if (!file || !canConvert.value) return

  try {
    converting.value = true
    file.status = 'converting'
    file.progress = 3
    file.serverMessage = '正在上传文件'

    const formData = new FormData()
    formData.append('file', file.raw)

    const response = await fetch(`${apiBaseUrl}/api/convert`, {
      method: 'POST',
      body: formData,
    })

    if (!response.ok) {
      throw new Error(await readErrorMessage(response))
    }

    applyServerProgress(file, await response.json())
    await pollProgress(file)
    converting.value = false
    completedFile.value = file
    showDownloadDialog.value = true
  } catch (error) {
    file.status = 'error'
    file.progress = 100
    file.serverMessage = error instanceof Error ? error.message : '转换失败'
    converting.value = false
    showNotice(file.serverMessage, 'destructive')
  }
}

function pollProgress(file: OfficeFile) {
  return new Promise<void>((resolve, reject) => {
    if (!file.taskId) {
      reject(new Error('转换任务创建失败'))
      return
    }

    clearProgressTimer()
    progressTimer = setInterval(async () => {
      try {
        const response = await fetch(`${apiBaseUrl}/api/convert/${file.taskId}/progress`)

        if (!response.ok) {
          throw new Error(await readErrorMessage(response))
        }

        const progress: ConvertProgress = await response.json()
        applyServerProgress(file, progress)

        if (progress.status === 'done') {
          clearProgressTimer()
          resolve()
        }

        if (progress.status === 'error') {
          clearProgressTimer()
          reject(new Error(progress.message || '转换失败'))
        }
      } catch (error) {
        clearProgressTimer()
        reject(error)
      }
    }, 500)
  })
}

function applyServerProgress(file: OfficeFile, progress: ConvertProgress) {
  file.taskId = progress.taskId
  file.progress = progress.progress
  file.serverMessage = progress.message
  file.downloadUrl = progress.downloadUrl
  file.status = progress.status === 'queued' ? 'converting' : progress.status
}

function clearProgressTimer() {
  if (progressTimer) {
    clearInterval(progressTimer)
    progressTimer = null
  }
}

async function readErrorMessage(response: Response) {
  try {
    const data = await response.json()
    return data.message || '请求失败'
  } catch {
    return '请求失败'
  }
}

function downloadFile(file: OfficeFile) {
  if (!file.downloadUrl) {
    showNotice('下载地址不存在，请重新转换', 'destructive')
    return
  }

  const link = document.createElement('a')
  link.href = `${apiBaseUrl}${file.downloadUrl}`
  link.download = `${file.name.replace(/\.[^.]+$/, '')}.pdf`
  link.click()
  showDownloadDialog.value = false
}

function showNotice(message: string, type: NoticeType = 'default') {
  notice.value = message
  noticeType.value = type

  if (noticeTimer) {
    clearTimeout(noticeTimer)
  }

  noticeTimer = setTimeout(() => {
    notice.value = ''
  }, 2600)
}
</script>

