<template>
  <management
    ref="role"
    :tableColumns="tableColumns"
    :data="{table: '/api/role/list' }"
    page-name="角色"
    add-button
    refresh-button
    @submit="submit"
    @delete="deleteRow"
    @refresh="search"
  >
    <Form
      :model="searchForm"
      inline
      :label-width="80"
      ref="searchForm"
      slot="searchForm"
    >
      <FormItem
        prop="name"
        label="角色名称"
      >
        <i-input
          type="text"
          v-model="searchForm.name"
          placeholder="输入角色名称搜索"
          style="width: 162px;"
        />
      </FormItem>
      <FormItem>
        <Button
          type="primary"
          @click="search"
        >查询</Button>
        <Button
          @click="$refs.searchForm.resetFields()"
          style="margin-left: 8px"
        >重置</Button>
      </FormItem>
    </Form>
    <Form
      :model="searchForm"
      inline
      :label-width="80"
      ref="modalForm"
      slot="modalForm"
    >
      <FormItem
        label="角色名称"
        prop="name"
      >
        <i-input
          type="text"
          v-model="modalForm.name"
          placeholder="角色名称"
          style="width: 162px;"
        />
      </FormItem>
      <FormItem
        label="角色描述"
        prop="description"
      >
        <i-input
          type="text"
          v-model="modalForm.description"
          placeholder="角色名称"
          style="width: 162px;"
        />
      </FormItem>
      <FormItem prop="activated" label="状态">
            <RadioGroup v-model="modalForm.activated">
              <Radio :label="1">启用</Radio>
              <Radio :label="0">停用</Radio>
            </RadioGroup>
          </FormItem>
    </Form>
  </management>
</template>

<script>
import management from "_c/management";
import { getInfo, setActivated } from "@/api/system-management";
export default {
  name: "role",
  components: {
    management
  },
  data() {
    return {
      searchForm: {
        name: ""
      },
      tableColumns: [
        {
          title: "id",
          key: "id",
          minWidth: 80,
          tooltip: true
        },
        {
          title: "角色名",
          key: "name",
          minWidth: 80,
          tooltip: true
        },
        {
          title: "角色描述",
          key: "description",
          minWidth: 100,
          tooltip: true
        },
        {
          title: "角色编号",
          key: "roleNo",
          minWidth: 100,
          tooltip: true
        },
        {
          title: "状态",
          key: "activated",
          minWidth: 80,
          tooltip: true,
          render: (h, { row }) => {
            return h("span", row.activated ? "启用" : "停用");
          }
        },
        {
          title: "权限",
          key: "createdBy",
          minWidth: 80,
          tooltip: true
        },
        {
          title: "最后修改人",
          key: "lastModifiedBy",
          minWidth: 100,
          tooltip: true
        },
        {
          title: "最后修改时间",
          key: "lastModifiedDate",
          minWidth: 110,
          tooltip: true
        },
        {
          title: "备注",
          key: "remark",
          minWidth: 100,
          tooltip: true
        },
        {
          title: "操作",
          key: "action",
          fixed: "right",
          width: 200,
          render: (h, { row }) => {
            return h("div", [
              h(
                "Button",
                {
                  props: {
                    type: "primary",
                    size: "small"
                  },
                  on: {
                    click: () => {
                      this.edit(row);
                    }
                  },
                  style: {
                    marginRight: "10px"
                  }
                },
                "编辑"
              ),
              h(
                "Button",
                {
                  props: {
                    type: "error",
                    size: "small"
                  },
                  on: {
                    click: () => {
                      this.$refs.role.delete(row);
                    }
                  },
                  style: {
                    marginRight: "10px"
                  }
                },
                "删除"
              ),
              h(
                "i-switch",
                {
                  props: {
                    size: "large",
                    value: row.activated,
                    loading: row.activated_loading
                  },
                  on: {
                    "on-change": value => {
                      this.changeActivated(row, value);
                    }
                  }
                },
                [
                  h("span", { slot: "open" }, "启用"),
                  h("span", { slot: "close" }, "停用")
                ]
              )
            ]);
          }
        }
      ],
      modalForm:{
        activated:"",
        createdBy: "",
        createdDate: "",
        deleted: false,
        description: "",
        id: null,
        lastModifiedBy: "",
        lastModifiedDate: "",
        menus: {},
        name: "",
        remark: "",
        roleNo: "",
      }
    };
  },
  methods: {
    search() {
      // 查询时，处理查询参数
      const param = Object.assign({}, this.searchForm);
      for (let i in param) {
        param[i] === "" && delete param[i];
      }
      this.$refs.role.setTableData(param);
    },
    submit(pid, request) {
      // 添加和保存时，参数的处理
      let param = Object.assign({}, this.modalForm);
      param.activated = !!param.activated;
      param.pid = pid;
      request("/api/role/", param, param.id ? "put" : "post");
    },
    deleteRow(request) {
      request("/api/role/");
    },
    edit({ id }) {
      // 编辑时，表单的回显
      getInfo("/api/role/info/" + id)
        .then(data => {
          Object.assign(this.modalForm, data, {
            activated: data.activated ? 1 : 0
          });
          this.$refs.organization.edit();
        })
        .catch(error => {
          this.$Notice.error({
            title: "打开编辑失败",
            desc: error
          });
        });
    },
    changeActivated(row, value) {
      row.activated = value;
      this.$set(row, "activated_loading", true);
      setActivated("/api/role/", row)
        .then(() => {
          this.$set(row, "activated_loading", false);
        })
        .catch(error => {
          row.activated = !value;
          this.$set(row, "activated_loading", false);
          this.$Notice.error({
            title: "状态修改失败",
            desc: error
          });
        });
    }
  }
};
</script>

<style>
</style>
