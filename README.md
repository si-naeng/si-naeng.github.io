====================================================================================
Step 2: Resolve “in process” issues by clearing stuck data
Connect to PostgreSQL and execute:
DO $$
DECLARE
    pid TEXT;
BEGIN
    FOR pid IN
        SELECT PROC_INST_ID_
        FROM ACT_RU_EXECUTION
        WHERE PROC_DEF_ID_ LIKE '%migConfiguration%'
    LOOP
        DELETE FROM ACT_RU_VARIABLE WHERE PROC_INST_ID_ = pid;
        DELETE FROM ACT_RU_TIMER_JOB WHERE PROCESS_INSTANCE_ID_ = pid;
        DELETE FROM ACT_RU_JOB WHERE PROCESS_INSTANCE_ID_ = pid;
        DELETE FROM ACT_RU_DEADLETTER_JOB WHERE PROCESS_INSTANCE_ID_ = pid;
        DELETE FROM ACT_RU_SUSPENDED_JOB WHERE PROCESS_INSTANCE_ID_ = pid;
        DELETE FROM ACT_RU_TASK WHERE PROC_INST_ID_ = pid;
        DELETE FROM ACT_RU_EXECUTION WHERE PROC_INST_ID_ = pid;
        RAISE NOTICE 'Cleaned MIG process: %', pid;
    END LOOP;
END;
$$;
Verify:
SELECT COUNT(*) FROM ACT_RU_EXECUTION
WHERE PROC_DEF_ID_ LIKE '%migConfiguration%';
-- Expected result: 0
