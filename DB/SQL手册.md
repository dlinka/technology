插入

    避免重复插入

        INSERT IGNORE INTO user(uid, username) VALUES(1, 'cr')
        
    如果存在则更新
    
        INSERT INTO user(uid, username) VALUES(1, 'cr') ON DUPLICATE KEY UPDATE username='CR'
        
    如果存在则删除再插入
    
        REPLACE INTO user(uid, username) VALUES(1, 'cr')
