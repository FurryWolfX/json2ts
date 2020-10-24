<template>
  <div>
    <a-row>
      <a-col :span="12">
        <a-textarea v-model="input" placeholder="请输入JSON或JavaClass" :auto-size="{ minRows: 30, maxRows: 30 }" />
      </a-col>
      <a-col :span="12">
        <a-textarea :value="output" placeholder="输出的interface" :auto-size="{ minRows: 30, maxRows: 30 }" readonly />
      </a-col>
    </a-row>
    <a-divider>参数</a-divider>
    <a-row>
      <a-col :span="24" style="text-align: center">
        <a-select default-value="JSON" style="width: 300px" size="large" @change="handleCodeTypeChange">
          <a-select-option value="JSON"> JSON </a-select-option>
          <a-select-option value="JAVA"> Java Class </a-select-option>
        </a-select>
        <a-input v-model="name" style="width: 300px" placeholder="请输入接口名称" size="large"> </a-input>
      </a-col>
    </a-row>
  </div>
</template>

<script lang="ts">
import { Component, Vue } from "vue-property-decorator";
import typeofJson from "typeof-jsonc/dist";

type CodeType = "JAVA" | "JSON";

@Component
export default class App extends Vue {
  private codeType: CodeType = "JSON";
  private name: string = "Result";
  private input: string = ``;

  get output(): string {
    if (this.codeType === "JAVA") {
      if (this.input) {
        try {
          const ast = this.parseJavaEntity(this.input);
          const props = ast.result.join("\n");
          let result = "";
          result += `export interface ${ast.className} {\n`;
          result += props;
          result += "\n}";
          return result;
        } catch (e) {
          return e.stack;
        }
      } else {
        return "";
      }
    } else {
      try {
        return typeofJson(this.input, this.name, {
          prefix: "",
          rootFlags: 1,
          disallowComments: false,
          addExport: true,
          singleLineJsDocComments: true,
        });
      } catch (e: Error) {
        return e.stack;
      }
    }
  }

  handleCodeTypeChange(type: CodeType) {
    this.codeType = type;
  }

  parseJavaEntity(code: string) {
    const resList: string[] = code.match(/private[ ]+[\w<>]+[ ]+[\w_0-9]+;/g);
    const result: string[] = [];
    resList.forEach((r) => {
      console.log(r);
      const res = /private[ ]+([\w<>]+)[ ]+([\w_0-9]+);/.exec(r);
      console.log("res", res);
      if (res && res.length) {
        result.push(`${res[2]}: ${this.parseJavaType(res[1])}`);
      }
    });

    const regClassName = /class (\w+)/;
    const resClassName = regClassName.exec(code);
    let className = "";
    if (resClassName && resClassName.length) {
      className = resClassName[1];
    }
    return {
      result,
      className,
    };
  }

  parseJavaType(type: string) {
    switch (type) {
      case "String":
        return "string;";
      case "Date":
        return "string; // Date";
      case "Integer":
        return "number;";
      case "int":
        return "number;";
      case "double":
        return "number;";
      case "float":
        return "number;";
      case "BigDecimal":
        return "number; // BigDecimal";
      case "bool":
        return "boolean;";
      default:
        return type.replace("List<", "Array<") + ";";
    }
  }
}
</script>

<style lang="scss" scoped>
.logo {
  text-align: center;
  img {
    height: 200px;
  }
}
.title {
  text-align: center;
}
</style>
