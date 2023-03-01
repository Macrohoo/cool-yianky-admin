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

    <cl-form ref="Form">
      <template #slot-refer="{ scope }">
        <el-table :data="scope.append" style="width: 100%" height="350">
          <el-table-column fixed prop="nickname" label="昵称" width="80" />
          <el-table-column prop="mobile_phone" label="手机号" width="120" />
          <el-table-column prop="chatgpt_type_cn" label="购买类型" width="80" />
          <el-table-column prop="chatgpt_vip_create_time" label="购买时间" />
        </el-table>
      </template>
    </cl-form>
  </cl-crud>
</template>

<script lang="tsx" setup name="crud">
import { useCrud, useUpsert, useTable, useForm } from "@cool-vue/crud";
import { service } from "/@/cool";
import utils from '../../../utils/index'

import { getCurrentInstance } from 'vue'
import { ElMessageBox, ElMessage } from 'element-plus'
const { appContext } = getCurrentInstance()!


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
      label: "是否受邀",
      prop: "refered_id",
      formatter(row) {
        return row.refered_id ? '受邀注册' : '自由注册'
      }
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
      width: 350,
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
                    url: '/admin/yianky/chatgpt/upgrade-vip',
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
        },
        {
          label: "分销详情",
          type: "success",
          onClick({ scope }) {
            console.log('scope______', scope)
            service.request({
              url: '/admin/yianky/chatgpt/refer-user',
              method: "POST",
              data: {
                id: scope.row.id
              }
            }).then(res => {
              console.log('res___________', res)
              scope.row.amount = String(res.referamount) + '元'
              scope.row.append = res.referUser.reduce((combined: any, currentVal: any) => {
                if (currentVal.chatgpt_type == 1) {
                  currentVal.chatgpt_type_cn = '日卡'
                }
                if (currentVal.chatgpt_type == 2) {
                  currentVal.chatgpt_type_cn = '周卡'
                }
                if (currentVal.chatgpt_type == 3) {
                  currentVal.chatgpt_type_cn = '月卡'
                }
                if (currentVal.chatgpt_type == 4) {
                  currentVal.chatgpt_type_cn = '年卡'
                }
                if (currentVal.chatgpt_type == 5) {
                  currentVal.chatgpt_type_cn = '永久卡'
                }
                currentVal.chatgpt_vip_create_time = utils.unixDate(currentVal.chatgpt_vip_create_time / 1000)
                combined.push(currentVal)
                return combined
              }, [])
              Form.value?.open({
                op: {
                  saveButtonText: '兑换'
                },
                title: "分销详情",
                items: [
                  {
                    label: "分销名单",
                    prop: 'append',    //如果component使用vm的写法，props也要写成append
                    value: scope.row.append,
                    component: {
                      name: "slot-refer"
                    }
                  },
                  {
                    label: "可兑换金额",
                    prop: 'amount',
                    value: scope.row.amount,
                    required: true,
                    component: {
                      name: "el-input",
                      props: {
                        disabled: true
                      }
                    }
                  }
                ],
                on: {
                  submit(data, { close, done }) {
                    ElMessageBox.confirm(
                      '确定后将兑换掉上面的所有分销数据',
                      '注意❗️提醒❗️',
                      {
                        confirmButtonText: '确定',
                        cancelButtonText: '取消',
                        type: 'warning',
                        draggable: true,
                      }
                    )
                      .then(() => {
                        ElMessage({
                          type: 'success',
                          message: '兑换提交',
                        })
                        let postData = {} as any
                        postData.superior = scope.row.id
                        postData.userIds = data.append.reduce((combied: any, currentVal: any) => {
                          combied.push(currentVal.id)
                          return combied
                        }, [])
                        //兑换分销（此次提交为止可见的数据）提交
                        service.request({
                          url: '/admin/refer-relation/add-redeemed',
                          method: "POST",
                          data: postData
                        }).then(res => {
                          console.log('Crud', Crud)
                          close()
                          Crud.value?.refresh()
                        })
                      })
                      .catch(() => {
                        ElMessage({
                          type: 'info',
                          message: '兑换取消',
                        })
                      })
                  }
                }
              })
            })
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
