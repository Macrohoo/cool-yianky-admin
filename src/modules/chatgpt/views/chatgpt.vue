<template>
  <cl-crud ref="Crud">
    <el-row>
      <cl-refresh-btn />

      <cl-flex1></cl-flex1>
      <cl-search-key />
    </el-row>

    <el-row>
      <cl-table ref="Table">
        <template #slot-btn="{ scope }">
          <el-button text bg>升级vip</el-button>
        </template>
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
import { proxy } from "/@/cool";
import utils from '../../../utils/index'

const Form = useForm();

const Crud = useCrud(
  {
    service: service.yianky.chatgpt
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
      label: "昵称",
      prop: "nickname"
    },
    {
      label: "创建时间",
      prop: "createTime"
    },
    {
      label: "手机",
      prop: "mobile_phone"
    },
    {
      label: "邮箱",
      prop: "email"
    },
    {
      label: "VIP类型",
      prop: "chatgpt_type",
      dict: [
        {
          label: "未购买",
          value: 0
        },
        {
          label: "日卡",
          value: 1
        },
        {
          label: "周卡",
          value: 2
        },
        {
          label: "月卡",
          value: 3
        },
        {
          label: "年卡",
          value: 4
        },
        {
          label: "永久",
          value: 5
        }
      ]
    },
    {
      label: "VIP生效时间",
      prop: "chatgpt_vip_create_time",
      formatter(row) {
        if (row.chatgpt_vip_create_time) {
          return utils.unixDate(row.chatgpt_vip_create_time / 1000)
        }
      }
    },
    {
      type: "op",
      width: 250,
      buttons: [
        "info",
        {
          label: "升级vip",
          type: "success",
          onClick({ scope }) {
            console.log('scope______', scope)
            // scope行数据
            Form.value?.open({
              title: "升级vip",
              items: [
                {
                  label: "昵称",
                  prop: 'nickname',
                  value: scope.row.nickname,
                  component: {
                    name: "el-input",
                    props: {
                      disabled: true
                    }
                  }
                },
                {
                  label: "用户ID",
                  prop: 'id',
                  value: scope.row.id,
                  required: true,
                  component: {
                    name: "el-input",
                    props: {
                      disabled: true
                    }
                  }
                },
                {
                  label: "VIP",
                  prop: 'chatgpt_type',
                  value: scope.row.chatgpt_type,
                  required: true,
                  component: {
                    name: "el-select",
                    options: [
                      {
                        label: '未购买',
                        value: '0'
                      },
                      {
                        label: '日卡',
                        value: '1'
                      },
                      {
                        label: '周卡',
                        value: '2'
                      },
                      {
                        label: '月卡',
                        value: '3'
                      },
                      {
                        label: '年卡',
                        value: '4'
                      },
                      {
                        label: '永久卡',
                        value: '5'
                      }
                    ]
                  }
                },
              ],
              on: {
                submit(data, { close, done }) {
                  const isDev = import.meta.env.MODE === "development"
                  delete data.nickname
                  data.vipLevel = data.chatgpt_type
                  delete data.chatgpt_type
                  //请求升级vip接口
                  service.request({
                    url: (isDev ? '' : '/api') + '/admin/yianky/chatgpt/upgrade-vip',
                    method: "POST",
                    data: data
                  }).then(res => {
                    console.log('Crud', Crud)
                    close()
                    Crud.value?.refresh()
                  })
                }
              }
            });
          }
        }
      ]
    }
  ]
});

//这是crud极速按钮处理
const Upsert = useUpsert({
  items: [
    {
      label: "姓名",
      prop: "nickname",
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
  //目前这里只开放info详情，不支持submit
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
