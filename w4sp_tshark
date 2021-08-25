do
    local gv = require('gv')

    -- 以下函数用于检查某个元素是否在一个表里
    -- 参考：http://stackoverflow.com/questions/2282444/how-to-check-if-a-
                table-contains-an-element-in-lua
    function table.contains(table, element)
        for _, value in pairs(table) do
            if value == element then
                return true
            end
        end
        return false
    end

    -- 构造一个TCP流对象
    local tcp_stream = Field.new("tcp.stream")

    -- 获得ip相关的几个对象，后续用于关系映射
    local eth_src = Field.new("eth.src")
    local ip = Field.new("ip")
    local ip_src = Field.new("ip.src")
    local ip_dst = Field.new("ip.dst")

    -- 做基本的服务分析
    local tcp = Field.new("tcp")
    local tcp_src = Field.new("tcp.srcport")
    local tcp_dst = Field.new("tcp.dstport")

    local udp = Field.new("udp")
    local udp_src = Field.new("udp.srcport")
    local udp_dst = Field.new("udp.dstport")

    --{ STREAMIDX:
    --    {
    --        SRCIP: srcip,
    --        DSTIP: dstip,
    --        SRCP:  srcport,
    --        DSTP:  dstport,
    --        TCP:    bool
    --    }
    --}

    streams = {}

    -- 用于创建监听条件（listenner）的函数
    local function init_listener()
        --不使用任何过滤器创建我们的listener，这样可以处理所有的帧
        local tap = Listener.new(nil, nil)



        --每个数据包都会执行以下调用
        function tap.packet(pinfo, tvb, root)
            local tcpstream = tcp_stream()
            local udp = udp() 
            local ip  = ip()

    

            if tcpstream then
                --查询streams表里记录过的tcp流编号，
                --如果这个编号的tcp流已经处理过，就直接返回
                if streams[tostring(tcpstream)] then
                    return 
                end
    

                --tcp流肯定有ip首部，调用tostring函数获得源和目标的IP及端口
                local ipsrc = tostring(ip_src())
                local ipdst = tostring(ip_dst())
                local tcpsrc = tostring(tcp_src())
                local tcpsrc = tostring(tcp_dst())

                --把流信息整合成一个表
                local streaminfo = {}
                streaminfo["ipsrc"] = ipsrc
                streaminfo["ipdst"] = ipdst
                streaminfo["psrc"] = tcpsrc
                streaminfo["pdst"] = tcpdst
                streaminfo["istcp"] = true
                streams[tostring(tcpstream)] = streaminfo
            end 

            if udp and ip then
                --udp流有ip首部，调用tostring函数获得源和目标的IP及端口
                local ipsrc = tostring(ip_src())
                local ipdst = tostring(ip_dst())
                local udpsrc = tostring(udp_src())
                local udpdst = tostring(udp_dst())

                --如果是“udp流”，
                --streams表里的键名（key）为ip:port:ip:port
                local udp_streama = ipsrc .. udpsrc .. ipdst .. udpdst
                local udp_streamb = ipdst .. udpdst .. ipsrc .. udpsrc

                --如果已经处理过了，返回
                --这个判断句确认没问题？

                if streams[udp_streama] or streams[udp_streamb] then
                    return 
                end

                local streaminfo = {}
                streaminfo["ipsrc"] = ipsrc
                streaminfo["ipdst"] = ipdst
                streaminfo["psrc"] = udpsrc
                streaminfo["pdst"] = udpdst
                streaminfo["istcp"] = false
                streams[udp_streama] = streaminfo
            end
        end		

        --只需要定义个空的tap.reset
        function tap.reset() 

        end

        function tap.draw()

            --创建一个graphviz元识图（unigraph）
            G = gv.graph("wireviz.lua")

            for k,v in pairs(streams) do 
                local streaminfo = streams[k]
                
                --为源端和目标端IP创建节点
                local tmp_s = gv.node(G, streaminfo["ipsrc"])
                local tmp_d = gv.node(G, streaminfo["ipdst"])

                --把节点连接起来
                local tmp_e = gv.edge(tmp_s, tmp_d)
                gv.setv(tmp_s, "URL", "")
                local s_tltip = gv.getv(tmp_s, "tooltip")
                local d_tltip = gv.getv(tmp_d, "tooltip")
                gv.setv(tmp_s, "tooltip", s_tltip .. "\n" .. streaminfo["psrc"])
        	      添if ["pdst"判断
                    gv.setv(tmp_d, "tooltip", d_tltip .. "\n" .. streaminfo["pdst"])

                if streaminfo["istcp"] then
                    gv.setv(tmp_e, "color", "red")
                else	
                    gv.setv(tmp_e, "color", "green")
                end
            end

            --gv.setv(G, "concentrate", "true")
            gv.setv(G, "overlap", "scale")
            gv.setv(G, "splines", "true")
            gv.layout(G, "neato")
            gv.render(G, "svg")
        -- tap.draw()函数结束
        end

    -- init_listener()函数结束
    end

    -- 调用init_listener函数
    init_listener()
end

