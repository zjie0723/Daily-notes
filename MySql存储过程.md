```

## MySql存储过程使用记录，
## 批量为不同的表新增字段，或者在某些表中新增一条记录。

/*************************************************更新部门表queryCode规则*************************************************/
DROP PROCEDURE IF EXISTS updateDepartQueryCode;

CREATE PROCEDURE updateDepartQueryCode(IN vParentId VARCHAR(50), IN vParentQueryCode VARCHAR(100))
BEGIN
  DECLARE done INT DEFAULT 0;

  DECLARE vDeptId VARCHAR(50) DEFAULT '';
  DECLARE vSourceId VARCHAR(50) DEFAULT '';

  DECLARE taskCursor CURSOR FOR SELECT id, sourceId FROM h_org_department t WHERE t.parentId = vParentId;
  DECLARE CONTINUE HANDLER FOR NOT FOUND SET done = 1;

  SET @@max_sp_recursion_depth = 300; /*设置递归最大深度*/

  OPEN taskCursor;

  REPEAT
    FETCH taskCursor INTO vDeptId, vSourceId;

    IF NOT done THEN

      /*SELECT vParentQueryCode;*/

      SET @vFullQueryCode = concat(vParentQueryCode, '#', vSourceId);

      /*SELECT @vFullQueryCode;*/

      /*更新queryCode，规则 parentQueryCode+'#'+sourceId */
      SET @sql1 = concat('update h_org_department d set d.queryCode = \'', @vFullQueryCode, '\' where d.id = \'', vDeptId, '\'');

      /*SELECT @sql1;*/

      PREPARE stmt1 FROM @sql1;
      EXECUTE stmt1;

      /*SELECT @vFullQueryCode;*/

      CALL updateDepartQueryCode(vDeptId, @vFullQueryCode);

    /*ELSE
      SELECT 'no child.';*/

    END IF;
  UNTIL done END REPEAT;
  CLOSE taskCursor;
END;
/***********************************************************************************************************/


/***************************************更新i表的queryCode***************************************************/
DROP PROCEDURE IF EXISTS updateITableQueryCode;

CREATE PROCEDURE updateITableQueryCode()
BEGIN
  DECLARE vTableName VARCHAR(50) DEFAULT '';
  DECLARE done INT DEFAULT 0;
  DECLARE vQueryCodeCount INT DEFAULT 0;

  DECLARE taskCursor CURSOR FOR SELECT table_name FROM information_schema.tables WHERE table_name like 'i_%' AND table_rows > 0;
  DECLARE CONTINUE HANDLER FOR NOT FOUND SET done = 1;
  OPEN taskCursor;

  REPEAT
    FETCH taskCursor INTO vTableName;
    IF NOT done THEN

      SET vQueryCodeCount = (SELECT count(*) FROM information_schema.columns WHERE table_name = vTableName AND column_name = 'ownerDeptQueryCode');
      IF vQueryCodeCount > 0 THEN

        /*SELECT vTableName;*/

        SET @alterSQL = concat('ALTER TABLE ', vTableName, ' MODIFY ownerDeptQueryCode varchar(512) default null');
        PREPARE stmt1 FROM @alterSQL;
        EXECUTE stmt1;

        SET @sql = concat('update ', vTableName, ' t set t.ownerDeptQueryCode = (select d.queryCode from h_org_department d where d.id = t.ownerDeptId)');

        /*SELECT @sql;*/

        PREPARE stmt FROM @sql;
        EXECUTE stmt;

      END IF;

    END IF;
  UNTIL done END REPEAT;
  CLOSE taskCursor;
END;

CALL updateITableQueryCode();
/***********************************************************************************************************/

/***********************************************************************************************************/
/*如果存在：则删除存储过程*/
DROP PROCEDURE IF EXISTS deleteBOAndWorkflow;


/*创建*/
CREATE PROCEDURE deleteBOAndWorkflow()
BEGIN
  DECLARE vWorkflowCode VARCHAR(50) DEFAULT ''; /*流程编码*/
  DECLARE vWorkflowInstanceId VARCHAR(50) DEFAULT ''; /*流程实例id*/
  DECLARE vBizObjectId VARCHAR(50) DEFAULT ''; /*业务对象id*/
  DECLARE vSchemaCode VARCHAR(50) DEFAULT ''; /*业务模型编码*/
  DECLARE vTableName VARCHAR(50) DEFAULT ''; /*业务表*/

  DECLARE done INT DEFAULT 0; /*定义游标结束标记*/

  DECLARE taskCursor CURSOR FOR select t.id, t.workflowCode, t.bizObjectId from biz_workflow_instance t;
  DECLARE CONTINUE HANDLER FOR NOT FOUND SET done = 1;

  OPEN taskCursor;
  REPEAT
    FETCH taskCursor INTO vWorkflowInstanceId, vWorkflowCode, vBizObjectId;

    IF NOT done THEN
      SET vSchemaCode = (select h.schemaCode from h_workflow_header h where h.workflowCode = vWorkflowCode);
      /*select vSchemaCode;*/

      IF vSchemaCode <> '' THEN
        SET vTableName = concat('i_', vSchemaCode);
        /*select vTableName;*/

        SET @tSQL = concat('select count(1) into @vTableCount from information_schema.tables where table_name = \'', vTableName, '\'');

        PREPARE tStmt FROM @tSQL;
        EXECUTE tStmt;
        DEALLOCATE PREPARE tStmt; /*释放资源*/

        /*select @vTableCount;*/
        /*判断表是否存在*/
        IF @vTableCount > 0 THEN

          SET @sql = concat('select count(*) into @vDataCount from ', vTableName, ' where id = \'', vBizObjectId, '\'');

          PREPARE stmt FROM @sql;
          EXECUTE stmt;
          DEALLOCATE PREPARE stmt;

          /*select @vDataCount;*/
          /*判断i表是否有数据*/
          IF @vDataCount = 0 THEN
             /*set vWorkflowInstanceId = 'aaaaa';*/
             /*select @vDataCount, vWorkflowInstanceId, vWorkflowCode, vBizObjectId, vSchemaCode;*/

             SET @sql1 = concat('delete from biz_workflow_instance where id = \'', vWorkflowInstanceId, '\'');
             SET @sql2 = concat('delete from biz_workflow_token where instanceId = \'', vWorkflowInstanceId, '\'');
             SET @sql3 = concat('delete from biz_workitem where instanceId = \'', vWorkflowInstanceId, '\'');
             SET @sql4 = concat('delete from biz_workitem_finished where instanceId = \'', vWorkflowInstanceId, '\'');
             SET @sql5 = concat('delete from biz_circulateitem where instanceId = \'', vWorkflowInstanceId, '\'');
             SET @sql6 = concat('delete from biz_circulateitem_finished where instanceId = \'', vWorkflowInstanceId, '\'');

             PREPARE stmtDel1 FROM @sql1;
             PREPARE stmtDel2 FROM @sql2;
             PREPARE stmtDel3 FROM @sql3;
             PREPARE stmtDel4 FROM @sql4;
             PREPARE stmtDel5 FROM @sql5;
             PREPARE stmtDel6 FROM @sql6;

             EXECUTE stmtDel1;
             EXECUTE stmtDel2;
             EXECUTE stmtDel3;
             EXECUTE stmtDel4;
             EXECUTE stmtDel5;
             EXECUTE stmtDel6;

             DEALLOCATE PREPARE stmtDel1;
             DEALLOCATE PREPARE stmtDel2;
             DEALLOCATE PREPARE stmtDel3;
             DEALLOCATE PREPARE stmtDel4;
             DEALLOCATE PREPARE stmtDel5;
             DEALLOCATE PREPARE stmtDel6;

          END IF;

        END IF;

      END IF;

    END IF;
  UNTIL done END REPEAT;

  CLOSE taskCursor;
END;

/*调用存储过程*/
CALL deleteBOAndWorkflow();
/***********************************************************************************************************/

/***********************************************************************************************************/
DROP PROCEDURE IF EXISTS addSortKeyForITable;

CREATE PROCEDURE addSortKeyForITable()
BEGIN
  DECLARE tableName VARCHAR(50) DEFAULT '';
  DECLARE done INT DEFAULT 0;
  DECLARE parentIdCount INT DEFAULT 0;
  DECLARE sortKeyCount INT DEFAULT 0;
  DECLARE taskCursor CURSOR FOR SELECT table_name FROM information_schema.tables WHERE table_name like 'i_%' AND table_rows > 0;
  DECLARE CONTINUE HANDLER FOR NOT FOUND SET done = 1;
  OPEN taskCursor;

  REPEAT
    FETCH taskCursor INTO tableName;
    IF NOT done THEN

      SET parentIdCount = (SELECT count(*) FROM information_schema.columns WHERE table_name = tableName AND column_name = 'parentId');
      IF parentIdCount > 0 THEN

        SET sortKeyCount = (SELECT count(*) FROM information_schema.columns WHERE table_name = tableName AND column_name = 'sortKey');
        IF sortKeyCount = 0 THEN

          SET @sql = concat('alter table ', tableName, ' add sortKey decimal(19,6) DEFAULT 0 COMMENT "子表排序号"');
          PREPARE stmt FROM @sql;
          EXECUTE stmt;

        END IF;

      END IF;

    END IF;
  UNTIL done END REPEAT;
  CLOSE taskCursor;
END;

CALL addSortKeyForITable();
/***********************************************************************************************************/

/***********************************************************************************************************/
DROP PROCEDURE IF EXISTS addSortKeyForITable;
CREATE PROCEDURE addSortKeyForITable()
BEGIN
  DECLARE schemaCode VARCHAR(50) DEFAULT '';
  DECLARE done INT DEFAULT 0;
  DECLARE sortKeyCount INT DEFAULT 0;
  DECLARE uuidKey VARCHAR(50) DEFAULT '';
  DECLARE taskCursor CURSOR FOR select code from h_biz_property t where t.propertyType = 'CHILD_TABLE';
  DECLARE CONTINUE HANDLER FOR NOT FOUND SET done = 1;
  OPEN taskCursor;
  REPEAT
    FETCH taskCursor INTO schemaCode;
    IF NOT done THEN
      SET sortKeyCount = (select count(*) from h_biz_property t where t.schemaCode = schemaCode and t.code = 'sortKey');
      IF sortKeyCount = 0 THEN

          SET uuidKey = REPLACE((select UUID()), '-', '');
          SET @sql = concat(
            'INSERT INTO h3bpm_dev.h_biz_property (id, createdTime, creater, deleted, modifiedTime, modifier, remarks, code, defaultProperty, defaultValue, name, propertyEmpty, propertyIndex, propertyLength, propertyType, published, relativeCode, schemaCode)',
            ' VALUES (\'', uuidKey, '\', \'2019-04-23 14:56:07\', null, false, \'2019-04-23 14:56:07\', null, null, \'sortKey\', true, null, \'子表排序号\', false, false, 10, \'NUMERICAL\', true, null, \'',
            schemaCode, '\')');
          PREPARE stmt FROM @sql;
          EXECUTE stmt;

      END IF;
    END IF;
  UNTIL done END REPEAT;
  CLOSE taskCursor;
END;

CALL addSortKeyForITable();
/***********************************************************************************************************/

```
