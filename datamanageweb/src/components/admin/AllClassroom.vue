<template>
  <n-data-table
      :columns="classroomColumns"
      :data="allClassroom"
      :pagination="paginationReactive"
      :bordered="false"
      :single-line="false"
      style="font-size: 15px;"
      striped
  />
</template>

<script>
import {NButton, NDataTable, useMessage, NSpace, NPopconfirm} from "naive-ui";
import {h, reactive, ref} from "vue";
import {useStore} from "vuex";
import $ from "jquery";

export default {
  components: {
    NDataTable,
  },
  setup() {
    const store = useStore();
    const message = useMessage();
    let allClassroom = ref([]);

    $.ajax({
      url: "https://data.lxcode.xyz/api/classroom/get-all/",
      type: "get",
      success(resp) {
        allClassroom.value = resp;
      }
    });

    const createClassroomColumns = ({
                                    deleteClassroom
                                  }) => {
      return [
        {
          title: "教室编号",
          key: "classroomId"
        },
        {
          title: "教室名称",
          key: "classroomName"
        },
        {
          title: "教室地址",
          key: "classroomSite"
        },
        {
          title: "操作",
          render(row) {
            const buttons = [
              {
                text: "删除教室",
                color: "error",
                onClick: () => deleteClassroom(row)
              }
            ];

            const handlePositiveClick = (row) => {
              return () => {
                helper(row);
              };
            };

            const helper = (row) => {
              $.ajax({
                url: "https://data.lxcode.xyz/api/classroom/delete/",
                type: "post",
                headers: {
                  Authorization: "Bearer " + store.state.user.token,
                },
                data: {
                  classroom_id: row.classroomId,
                },
                success(resp) {
                  if (resp.error_message === "success") {
                    message.success("删除成功！");
                    setTimeout(() => { location.reload(); }, 800)
                  } else {
                    message.error(resp.error_message);
                  }
                }
              });
            }

            const handleNegativeClick = () => {};

            return h(
                "div",
                {},
                [
                  h(
                      NSpace,
                      { align: "center" },
                      buttons.map(({ text, color, onClick }, index) =>
                          h(
                              NPopconfirm,
                              {
                                onPositiveClick: handlePositiveClick(row),
                                onNegativeClick: handleNegativeClick,
                                key: index
                              },
                              {
                                trigger: () =>
                                    h(
                                        NButton,
                                        {
                                          strong: true,
                                          type: color,
                                          size: "small",
                                          onClick
                                        },
                                        { default: () => text }
                                    ),
                                default: () =>
                                    h("span", {}, "您确定要删除该教室吗？")
                              }
                          )
                      )
                  )
                ]
            );
          }
        }



      ];
    };

    const paginationReactive = reactive({
      page: 2,
      pageSize: 7,
      showSizePicker: true,
      pageSizes: [3, 5, 7, 9, 11],
      onChange: (page) => {
        paginationReactive.page = page;
      },
      onUpdatePageSize: (pageSize) => {
        paginationReactive.pageSize = pageSize;
        paginationReactive.page = 1;
      }
    });


    return {
      paginationReactive,
      allClassroom,
      classroomColumns: createClassroomColumns({
        deleteClassroom() {}
      })
    }
  }
}

</script>

<style scoped>

</style>