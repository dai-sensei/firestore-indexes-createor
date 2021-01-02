<template>
  <div>
    <div class="collection-name">
      コレクション名（必須）
      <div>
        <input
            type="text"
            v-model="collectionName"
            class="name"
        />
      </div>
      <span
          v-if="!isValidCollectionName"
          class="alert-text"
      >
        コレクション名は、入力必須です
      </span>
    </div>
    <div class="query-scope">
      クエリスコープ
      <div>
        <select v-model="queryScope">
          <option value="COLLECTION">コレクション</option>
          <option value="COLLECTION_GROUP">コレクショングループ</option>
        </select>
      </div>
    </div>
    <div class="field">
      フィールド(フィールド名 | ASCENDING or DESCENDING)
      <div>
        <textarea
            v-model="fieldNames"
            rows="10"
            cols="60"
        />
      </div>
    </div>
    <div class="field">
      固定のフィールド(フィールド名 | ASCENDING or DESCENDING)
      <div>
        <textarea
            v-model="fixedFieldNames"
            rows="10"
            cols="60"
        />
      </div>
    </div>
    <div>
      <button id="create-index" @click="create">index生成</button>
    </div>
    <div class="field">
      <div>index一覧</div>
      <textarea
          readonly
          v-model="indexes"
          rows="20"
          cols="60"
      />
      <h3>総パターン数: {{patternCount}}</h3>
    </div>
  </div>
</template>

<script>
import { combinations } from 'iter-tools/es2015';

export default {
  name: 'ImportForm',
  data() {
    return {
      collectionName: '',
      queryScope: 'COLLECTION_GROUP',
      fieldNames: '',
      fixedFieldNames: '',
      indexes: '',
      patternCount: 0,
      isValidCollectionName: true
    };
  },
  methods: {
    create() {
      this.indexes = '';
      this.isValidCollectionName = !!this.collectionName;

      if (!this.isValidCollectionName) return;

      // 改行文字で配列に分割
      const fieldNames = this.fieldNames.split('\n').filter(v => !!v);

      // 改行文字で配列に分割
      const fixedFieldNames = this.fixedFieldNames.split('\n').filter(v => !!v);

      // フィールド数
      let fieldCount = fieldNames.length;

      // 処理終了
      if (fieldCount === 0) return;

      const list = [];

      for (let i = 1; i <= fieldCount; i++) {
        // iは選択するfield数
        const patterns = this.createPatterns(i, this.collectionName, this.queryScope, fieldNames, fixedFieldNames);
        this.patternCount += patterns.length;
        list.push(...patterns);
      }

      this.indexes = JSON.stringify(list, null, "  ");
    },
    createPatterns(selectedNum, collectionName, queryScope, fieldNames, fixedFieldNames) {
      const list = [];

      if (selectedNum === 1) {
        for (const fieldName of fieldNames) {
          const indexData = [];
          const field = fieldName.split('|');

          indexData.push({
            fieldPath: field[0],
            order: field[1] ? field[1] : "ASCENDING"
          });

          // 固定値の追加
          for (const fixedFieldName of fixedFieldNames) {
            const field = fixedFieldName.split('|');

            indexData.push({
              fieldPath: field[0],
              order: field[1] ? field[1] : "ASCENDING"
            });
          }
          list.push({
            collectionGroup: collectionName,
            queryScope: queryScope,
            fields: indexData
          });
        }
        return list;
      }

      const combinationPatterns = combinations(fieldNames, selectedNum);

      // 複数選択のパターン
      for (const pattern of combinationPatterns) {

        const indexData = [];

        for (const index of pattern) {
          const field = index.split('|');

          indexData.push({
            fieldPath: field[0],
            order: field[1] ? field[1] : "ASCENDING"
          });
        }

        // 固定値の追加
        for (const fixedFieldName of fixedFieldNames) {
          const field = fixedFieldName.split('|');

          indexData.push({
            fieldPath: field[0],
            order: field[1] ? field[1] : "ASCENDING"
          });
        }

        list.push({
          collectionGroup: collectionName,
          queryScope: queryScope,
          fields: indexData
        });
      }
      return list;
    }
  }
}
</script>

<style scoped>
.field {
  margin-top: 20px;
}
.query-scope {
  margin-top: 20px;
}
.name {
  width: 50%;
}
.alert-text {
  color: red;
  font-size: 0.8rem;
}
textarea {
  width: 50%;
}
#create-index {
  width: 50%;
}
</style>
