/*入库记录管理,作者:alvin198761@163.com,日期:2019-11-17 21:00:17*/
<template>
      <div>
            <Form inline :label-width="70"  style="text-align: left">
                                                                                                        <FormItem label="产品id" prop="prodId">
                                                        <Input placeholder="请输入产品id" v-model="form.prodId"/>
                                                        </FormItem>
                                                                                                    <FormItem label="入库日期" prop="date">
                                                        <DatePicker type="date" placeholder="请选择入库日期" v-model="form.date" clearable></DatePicker>
                                                        </FormItem>
                                                                                                    <FormItem label="发票类型" prop="ticketType">
                                                        <Input placeholder="请输入发票类型" v-model="form.ticketType"/>
                                                        </FormItem>
                                                                                                    <FormItem label="发票状态" prop="ticketStatus">
                                                        <Input placeholder="请输入发票状态" v-model="form.ticketStatus"/>
                                                        </FormItem>
                                                                                                    <FormItem label="备注" prop="remark">
                                                        <Input placeholder="请输入备注" v-model="form.remark"/>
                                                        </FormItem>
                                                                          <FormItem  :label-width="0">
                        <Button icon="search" @click="refresh" style="margin-right: 5px" title="根据输入的条件查询" type="primary">查询</Button>

                        <Button type="primary" icon="plus" style="margin-right: 5px" @click="doAdd()" title="添加" >添加</Button>
                      </FormItem>
                </Form>
            <Table :loading="loading" :columns="tableHeader" :data="dataList" style="width: 100%">
                          <template slot-scope="{ row,index }" slot="col_id">
                <div>{{ row.id }}</div>
              </template>
                          <template slot-scope="{ row,index }" slot="col_prodId">
                <div>{{ row.prodId }}</div>
              </template>
                          <template slot-scope="{ row,index }" slot="col_date">
                <div>{{ row.date }}</div>
              </template>
                          <template slot-scope="{ row,index }" slot="col_ticketType">
                <div>{{ row.ticketType }}</div>
              </template>
                          <template slot-scope="{ row,index }" slot="col_ticketStatus">
                <div>{{ row.ticketStatus }}</div>
              </template>
                          <template slot-scope="{ row,index }" slot="col_remark">
                <div>{{ row.remark }}</div>
              </template>
                        <template slot-scope="{ row,index }" slot="col_opt">
              <div>
              <Button type="primary" size="small" style="margin-right: 5px" @click="doEdit(row)">编辑</Button>
              <Button type="primary" size="small"   style="margin-right: 5px" @click="doDelete(row)">删除</Button>
              </div>
            </template>
            </Table>
            <br/>
            <div style="text-align: right" v-if="total > 0">
              <Page size="small" :current="page" :total="total" show-total  @on-change="(curr) => {this.page = curr ; this.refresh();}"
              show-sizer @on-page-size-change="(pageSize) => { this.pageSize = pageSize ; this.refresh();}" :page-size="pageSize"  ></Page>
            </div>
        <InboundDialog ref="dialog" :refresh="refresh"></InboundDialog>
  </div>
</template>
<script>
import InboundDialog from './InboundDialog.vue';
import InboundListExpand from './InboundListExpand.vue';
  export default {
    components: {
     InboundDialog,
     InboundListExpand
   },
    data: function () {
      const that = this;
      return {
        tableHeader: [
          { type: 'expand',  width: 50, render: (h, params) => { return h(InboundListExpand, {  props: {row: params.row}  })  }  },
                            {title: '主键', key: 'id', slot:'col_id' },
                            {title: '产品id', key: 'prodId', slot:'col_prodId' },
                            {title: '入库日期', key: 'date', slot:'col_date' },
                            {title: '发票类型', key: 'ticketType', slot:'col_ticketType' },
                            {title: '发票状态', key: 'ticketStatus', slot:'col_ticketStatus' },
                            {title: '备注', key: 'remark', slot:'col_remark' },
                    {            title: '操作',            width: 150,            slot : 'col_opt'          }
        ],
        total: 0,
        page: 1,
        pageSize: 20,
        dataList: [],
        form: {
                            id : null,// 主键
                            prodId : null,// 产品id
                            date : null,// 入库日期
                            ticketType : null,// 发票类型
                            ticketStatus : null,// 发票状态
                            remark : null,// 备注
                    },
        loading: false
      }
    },
    computed: {},
    created: function () {
      this.refresh();
    },
    methods: {
      refresh() {
        const that = this;
        that.loading = true;
        const requestData = {...that.form, page: that.page - 1,size:that.pageSize};
        that.$postBody("/api/inbound/queryPage", requestData).then(res => {
              that.loading = false;
              if (res.errcode != 0) {
                that.$Message.error({content: res.errmsg ? res.errmsg : '获取入库记录列表失败'});
                return ;
              }
		          that.dataList = res.replydata.content;
		          that.total = res.replydata.totalElements;
		        }).catch(res => {
		          that.$Message.error({ content: '获取入库记录列表失败：' + res });
		        });
      },
      doAdd() {
        this.$refs["dialog"].addDialog();
      },
      doEdit(row) {
        this.$refs["dialog"].editDialog(row);
      },
      doDelete(row) {
        const that = this;
        that.$Modal.confirm({
          title: '提示',
          content: '您确定要删除吗?',
          closable:true,
          onOk: () => {
            this.$post('/api/inbound/delete', {id: row.id }).then(res => {
                  if (res.errcode != 0) {
                    that.$Message.error({content: res.errmsg ? res.errmsg : '获取入库记录列表失败'});
                    return ;
                  }
  		            that.$Message.success({ content: '删除成功' });
  		            that.refresh();
  		          }).catch(res => {
  		            that.$Message.error({ content: '删除失败：' + res });
  		          });
          },
          onCancel: () => {
          }
        });
      }
    }
  }
</script>
<style>
  .expand-row {
    margin-bottom: 6px;
  }
  .expand-key {
    font-weight: bold;
    line-height: 25px;
  }
  .expand-value {}
</style>