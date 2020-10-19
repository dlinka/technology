1.进入MessageClientExt

    @Override
    public String getMsgId() {
        String uniqID = MessageClientIDSetter.getUniqID(this);
        if (uniqID == null) {
            return this.getOffsetMsgId();
        } else {
            return uniqID;
        }
    }
