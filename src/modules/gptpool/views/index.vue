<template>
  <cl-crud ref="Crud">
    <el-row>
      <cl-refresh-btn />
      <cl-add-btn />
      <span style="text-align: center; line-height: 32px; padding-left: 30px; color: brown;">更改数据后，客户端生效需要5分钟</span>

      <cl-flex1></cl-flex1>
      <cl-search-key />
    </el-row>

    <el-row>
      <cl-table ref="Table">
      </cl-table>
    </el-row>

    <el-row>
      <cl-flex1></cl-flex1>
      <cl-pagination></cl-pagination>
    </el-row>

    <cl-upsert ref="Upsert">
    </cl-upsert>

    <cl-form ref="Form"></cl-form>
  </cl-crud>
</template>

<script lang="tsx" setup name="crud">
import { useCrud, useUpsert, useTable, useForm } from "@cool-vue/crud";
import { service } from "/@/cool";

const Form = useForm();

const Crud = useCrud(
  {
    service: service.yianky.gptpool
  },
  (app) => {
    app.refresh();
  }
);


//这里包含定制按钮
const Table = useTable({
  columns: [
    {
      type: "selection"
    },
    {
      label: "apikey",
      prop: "api_key",
      width: 600
    },
    {
      label: "伪造请求ip",
      prop: "ip"
    },
    {
      type: "op",
      width: 250,
      buttons: ["info", "edit", "delete"]
    }
  ]
});

//这是crud极速按钮处理
const Upsert = useUpsert({
  items: [
    {
      label: "apikey",
      prop: "api_key",
      required: true,
      component: {
        name: "el-input"
      }
    },
    {
      label: "请求伪造ip",
      prop: "ip",
      required: true,
      component: {
        name: "el-input"
      }
    }
  ],
  onInfo(data, { next }) {
    console.log(data);
    next(data);
    // done({ name: "🐑" });
    // close();
  },
  onSubmit(data, { next }) {
    console.log(data);
    next(data);
    // Upsert.value?.close();
  },
  onOpen(data) {
    console.log(data);
  },
  onClose(done) {
    console.log("onclose");
    done();
  }
});
</script>
