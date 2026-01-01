<script lang="ts">
  import dayjs from "dayjs";
  import { onMount } from "svelte";
  import { coreEndpointsGetStatus, type OutSchemaStr, type ErrorDetailSchema, type coreEndpointsGetStatusResponse } from "./lib/api/backend-api";

  let statusMessage: string = "正在请求后端……";
  let statusData: string | null = null;
  let statusErrors: ErrorDetailSchema | null = null;
  let statusTimestamp: string | null = null;

  onMount(() => {
    coreEndpointsGetStatus()
      .then((response: coreEndpointsGetStatusResponse) => {
        const data: OutSchemaStr = response.data;

        statusMessage = data.message || "未知状态";
        statusData = data.data || null;

        if (data.errors && Array.isArray(data.errors) && data.errors.length > 0) {
          statusErrors = data.errors[0];
        } else {
          statusErrors = null;
        }

        statusTimestamp = data.timestamp ? dayjs(data.timestamp).format("YYYY年MM月DD日HH时mm分ss秒") : null;
      })
      .catch((error: Error) => {
        statusMessage = "error";
        statusErrors = {
          field: "network",
          message: `${error.message}`,
        };
        statusTimestamp = dayjs().format("YYYY年MM月DD日HH时mm分ss秒");
      });
  });
</script>

<p class="text-2xl">
  <span class="font-bold">消息：</span>
  <span class="underline">{statusMessage}</span>
</p>

{#if statusData}
  <p class="text-2xl">
    <span class="font-bold">数据：</span>
    <span class="underline">{statusData}</span>
  </p>
{/if}

{#if statusErrors}
  <p class="text-2xl text-red-500">
    <span class="font-bold">错误：</span>
    <span class="underline">{statusErrors.message}</span>
    {#if statusErrors.field}
      <span class="font-bold">字段：</span>
      <span class="underline">{statusErrors.field}</span>
    {/if}
  </p>
{/if}

{#if statusTimestamp}
  <p class="text-2xl">
    <span class="font-bold">时间：</span>
    <span class="underline">{statusTimestamp}</span>
  </p>
{/if}
