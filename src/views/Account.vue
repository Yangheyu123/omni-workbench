<template>
  <main class="account">
    <h1>账户中心(演示)</h1>
    <div class="cards">
      <div class="card">
        <span>可用余额</span>
        <strong>{{ available }}</strong>
        <small v-if="frozen > 0">冻结中 {{ frozen }}(生成完成后结算)</small>
      </div>
      <div class="card hint">
        <p>计费流程与生产一致:提交生成 → 按报价<b>冻结</b> → 完成<b>结算</b> / 失败<b>释放</b>。本演示中金额为模拟数值。</p>
        <el-button type="primary" plain @click="$router.push('/create?episode_id=1')">返回工作台</el-button>
      </div>
    </div>
    <h2>交易流水</h2>
    <el-table :data="transactions" stripe>
      <el-table-column prop="created_at" label="时间" width="200">
        <template #default="{ row }">{{ formatTime(row.created_at) }}</template>
      </el-table-column>
      <el-table-column prop="type" label="类型" width="120">
        <template #default="{ row }">
          <el-tag :type="row.type === 'settlement' ? 'success' : row.type === 'authorization' ? 'warning' : row.type === 'void' ? 'danger' : 'info'" size="small">
            {{ { authorization: '预冻结', settlement: '结算', void: '失败释放', recharge: '充值' }[row.type] || row.type }}
          </el-tag>
        </template>
      </el-table-column>
      <el-table-column prop="amount" label="金额(积分)" width="130" />
      <el-table-column prop="summary" label="说明" min-width="260" />
    </el-table>
  </main>
</template>

<script setup>
import { computed, onMounted, ref } from 'vue'
import { accountAPI } from '@/api/account'
import { formatChinaDateTime } from '@/utils/time'

const account = ref({ available: 0, frozen: 0 })
const transactions = ref([])
const available = computed(() => Number(account.value.available ?? 0).toLocaleString('zh-CN', { maximumFractionDigits: 4 }))
const frozen = computed(() => Number(account.value.frozen ?? 0))
const formatTime = formatChinaDateTime

onMounted(async () => {
  const [me, tx] = await Promise.all([accountAPI.me(), accountAPI.transactions({ page: 1, page_size: 50 })])
  account.value = me || {}
  transactions.value = tx?.items || []
})
</script>

<style scoped>
.account { max-width: 880px; margin: 40px auto; padding: 0 24px; }
.cards { display: flex; gap: 16px; margin: 18px 0 28px; flex-wrap: wrap; }
.card { border: 1px solid var(--border-color, #ddd); border-radius: 12px; padding: 18px 22px; min-width: 220px; background: var(--bg-card, #fafafa); }
.card span { display: block; color: #888; font-size: 13px; margin-bottom: 6px; }
.card strong { font-size: 28px; color: #409eff; }
.card.hint { flex: 1; min-width: 280px; font-size: 13px; color: #666; display: flex; flex-direction: column; gap: 10px; justify-content: center; }
</style>
