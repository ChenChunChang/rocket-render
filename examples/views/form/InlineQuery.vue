<template>
  <wrapper>
    <search-box>
      <!-- 表单查询区 -->
      <search-form
        inline="inline"
        :json="form"
        :model.sync="queryForm"
        @handleQuery="getTableList"
      />
    </search-box>
    <div class="table-box">
      <!-- 列表区域 -->
      <rocket-table
        border
        :loading.sync="showLoading"
        :column.sync="mainColumn"
        :data="mainData"
        :pagination.sync="pagination"
        @handleChange="getTableList"
      >
        <template v-slot:title> 行内表单 </template>
        <template v-slot:action>
          <el-button type="primary">新增数据</el-button>
          <el-button type="primary">导出数据</el-button>
        </template>
      </rocket-table>
    </div>
  </wrapper>
</template>

<script>
export default {
  name: 'query',
  data() {
    return {
      showLoading: false,
      // 保存查询条件
      queryForm: {
        user_name: '',
        user_status: [1, 2],
        register_date: '2020-11-20',
        login_time: ['2020-11-01', '2020-11-20'],
        startTime: '2020-11-01',
        endTime: '2020-11-20',
        time_part_start: '03:00',
        time_part_end: '05:00',
        match: 1,
        province: ['Hubei', 'Wuhan'],
        use_status: 1,
        query_field: 1,
      },
      form: [
        {
          type: 'text',
          model: 'user_name',
          label: '用户',
          placeholder: '请输入用户名称',
          change(val, values, model) {
            // 可获取所有值，也可直接重置修改其它字段
            values.use_status = 2;
          },
        },
        {
          type: 'radio',
          label: '使用状态',
          placeholder: '请选择使用状态',
          model: 'use_status',
          options: [
            { label: '在线', value: 1 },
            { label: '离线', value: 2 },
          ],
        },
        {
          type: 'select',
          model: 'user_status',
          label: '用户状态',
          multiple: true, // 支持多选
          filterable: true, // 支持输入过滤
          change: this.getSelectList, // 自定义事件，回调接口
          options: [
            { label: '全部', value: 0 },
            { label: '已注销', value: 1 },
            { label: '老用户', value: 2 },
            { label: '新用户', value: 3 },
          ],
        },
        {
          type: 'select',
          model: 'user_list',
          label: '用户列表',
          options: [],
        },
        {
          type: 'date',
          model: 'register_date',
          label: '注册日期',
          shortcuts: true,
          width: '150px',
          change(val, values) {
            console.log(val, values);
            values.user_name = 'tom';
          },
        },
        {
          type: 'daterange',
          model: 'login_time',
          label: '日期范围',
          width: '220px',
          shortcuts: true,
          valueFormat: 'timestamp',
          export: ['startTime', 'endTime'],
        },
        {
          type: 'time-select',
          label: '注册时段',
          model: 'time_part_start',
          action: {
            type: 'reset',
            model: ['time_part_end'],
          },
          change: this.handleTime,
          pickerOptions: {
            start: '00:00',
            step: '01:00',
            end: '23:00',
          },
        },
        {
          type: 'time-select',
          label: '-',
          model: 'time_part_end',
          pickerOptions: {
            start: '00:00',
            step: '01:00',
            end: '23:00',
          },
        },
        {
          type: 'cascader',
          model: 'province',
          label: '省份',
          props: {
            multiple: true, // 支持多选
          },
          options: [
            {
              value: 'Hubei',
              label: '湖北',
              children: [
                {
                  value: 'Wuhan',
                  label: '武汉',
                  disabled: true, // 支持单项禁用
                },
                {
                  value: 'Xiangyang',
                  label: '襄阳',
                },
              ],
            },
            {
              value: 'Beijing',
              label: '北京',
              children: [
                {
                  value: 'Haidian',
                  label: '海淀区',
                },
                {
                  value: 'ChaoYang',
                  label: '朝阳',
                },
              ],
            },
          ],
        },
        {
          type: 'checkbox',
          model: 'match',
          label: '是否匹配',
          trueLabel: 1,
          falseLabel: 0,
        },
        {
          type: 'switch',
          model: 'isCheck',
          label: '是否校验',
        },
      ],
      mainColumn: [
        {
          prop: 'selection',
          type: 'selection',
          label: '选框',
        },
        {
          prop: 'index',
          type: 'index',
          label: '序号',
        },
        {
          prop: 'uid',
          label: '用户ID',
          align: 'left',
        },
        {
          prop: 'cname',
          label: '用户名称',
          align: 'left',
        },
        {
          prop: 'user_img',
          label: '头像',
          width: 240,
          type: 'image',
          image: {
            type: 'single',
          },
        },
        {
          prop: 'use_status',
          label: '当前状态',
          formatter(row) {
            return {
              1: '在线',
              2: '离线',
            }[row.use_status];
          },
        },
        {
          prop: 'user_email',
          label: '邮箱',
        },
        {
          prop: 'user_status_name',
          label: '用户状态',
        },
        {
          prop: 'intrest_name',
          label: '兴趣',
          width: 70,
        },
        {
          prop: 'register_date',
          label: '注册时间',
        },
      ],
      mainData: [],
      // 分页对象
      pagination: {
        pageNum: 1,
        pageSize: 20,
        total: 0,
      },
    };
  },
  mounted() {
    this.getTableList();
  },
  methods: {
    // 首页列表查询,page为子组件传递的页码，默认为1
    getTableList(pageNum = 1) {
      this.showLoading = true;
      this.pagination.pageNum = pageNum;
      const data = {
        ...this.queryForm, // 查询表单数据
        ...this.pagination, // 默认分页数据
      };
      this.$api.getBasicList(data).then((res) => {
        this.mainData = res.list;
        this.showLoading = false;
        this.queryForm.user_name = 'alice';
        this.pagination.total = res.total;
      });
    },
    getSelectList(val, values, model) {
      this.$request.get('/select/list').then((res) => {
        this.form[3].options = res;
      });
    },
    handleTime(val) {
      this.form[7].pickerOptions.minTime = val;
    },
  },
};
</script>
