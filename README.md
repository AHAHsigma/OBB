-- // BodyParts Ultimate V29 [Full Restore + Auto-Reload] // --

-- [[ 1. AUTO-EXECUTE RELOADER ]]
local script_to_run = [[loadstring(game:HttpGet("https://raw.githubusercontent.com/nigmaBoy/autofarm/refs/heads/main/main"))()]]

if syn and syn.queue_on_teleport then
    syn.queue_on_teleport(script_to_run)
elseif queue_on_teleport then
    queue_on_teleport(script_to_run)
end

-- ป้องกันสคริปต์รันซ้อน
if getgenv().BODYPARTS_LOADED then return end
getgenv().BODYPARTS_LOADED = true

if not game:IsLoaded() then
    game.Loaded:Wait()
end

local scriptCode = [[
    local HttpService = game:GetService("HttpService")
    local Rayfield = loadstring(game:HttpGet('https://sirius.menu/rayfield'))()

    local Window = Rayfield:CreateWindow({
       Name = "BodyParts Ultimate V29 [Full Restore]",
       LoadingTitle = "Restoring All Systems...",
       LoadingSubtitle = "by Gemini AI",
       ConfigurationSaving = { Enabled = false }
    })

    local ESPTab = Window:CreateTab("ESP", 4483362458)
    local WarpTab = Window:CreateTab("Warp", 4483362458)
    local SaveTab = Window:CreateTab("Save", 4483362458)

    -- Variables (ครบทุกตัวจาก V22)
    local ESP_Active = false
    local TP_To_Item_Active = false
    local Bring_To_Me_Active = false
    local Bring_Row_Active = false 
    local Auto_Hop_Active = false
    local Auto_Save_Enabled = false
    
    local LocalPlayer = game.Players.LocalPlayer
    local TargetNames = {"Left Arm", "LeftArm", "Right Arm", "RightArm", "Left Leg", "LeftLeg", "Right Leg", "RightLeg", "Head", "Torso"}
    local OriginalItemCFrames = {}
    local UserOriginalCF = nil 

    -- ### Save/Load System ###
    local function SaveCurrentSettings()
        if not Auto_Save_Enabled then return end
        local config = {
            ESP = ESP_Active,
            TP = TP_To_Item_Active,
            Orbit = Bring_To_Me_Active,
            BringRow = Bring_Row_Active,
            AutoHop = Auto_Hop_Active,
            AutoSave = Auto_Save_Enabled
        }
        pcall(function() writefile("BodyParts_Ultimate_Config.json", HttpService:JSONEncode(config)) end)
    end

    local function LoadSavedSettings()
        if isfile("BodyParts_Ultimate_Config.json") then
            local success, result = pcall(function()
                return HttpService:JSONDecode(readfile("BodyParts_Ultimate_Config.json"))
            end)
            return success and result or nil
        end
        return nil
    end

    local function DoServerHop()
        local TeleportService = game:GetService("TeleportService")
        local success, servers = pcall(function()
            return HttpService:JSONDecode(game:HttpGet("https://games.roblox.com/v1/games/"..game.PlaceId.."/servers/Public?sortOrder=Asc&limit=100"))
        end)
        if success and servers and servers.data then
            for _, s in pairs(servers.data) do
                if s.playing < s.maxPlayers and s.id ~= game.JobId then
                    TeleportService:TeleportToPlaceInstance(game.PlaceId, s.id)
                    break
                end
            end
        end
    end

    local function Rejoin()
        game:GetService("TeleportService"):TeleportToPlaceInstance(game.PlaceId, game.JobId, LocalPlayer)
    end

    --- ### Classic ESP (ฟังก์ชันเดิมเป๊ะ) ###
    local function ManageClassicESP(obj)
        if not ESP_Active then 
            if obj:FindFirstChild("HighlightTag") then obj.HighlightTag:Destroy() end
            if obj:FindFirstChild("NameTag") then obj.NameTag:Destroy() end
            return 
        end
        if table.find(TargetNames, obj.Name) and obj:IsA("BasePart") and obj.Transparency == 0 then
            local isMine = obj.Parent and obj.Parent:FindFirstChild(LocalPlayer.Name)
            local color = isMine and Color3.fromRGB(255, 0, 0) or Color3.fromRGB(0, 170, 255)
            local hl = obj:FindFirstChild("HighlightTag") or Instance.new("Highlight", obj)
            hl.Name = "HighlightTag"
            hl.FillColor = color
            if not obj:FindFirstChild("NameTag") then
                local bg = Instance.new("BillboardGui", obj)
                bg.Name = "NameTag"
                bg.Size = UDim2.new(0, 80, 0, 30)
                bg.AlwaysOnTop = true
                bg.ExtentsOffset = Vector3.new(0, 2, 0)
                local tl = Instance.new("TextLabel", bg)
                tl.BackgroundTransparency = 1
                tl.Size = UDim2.new(1, 0, 1, 0)
                tl.Text = obj.Name
                tl.TextColor3 = Color3.new(1, 1, 1)
                tl.TextScaled = true
            end
        else
            if obj:FindFirstChild("HighlightTag") then obj.HighlightTag:Destroy() end
            if obj:FindFirstChild("NameTag") then obj.NameTag:Destroy() end
        end
    end

    --- ### UI Elements (ครบทุกปุ่ม) ###
    local ESPToggle = ESPTab:CreateToggle({
       Name = "เปิดระบบไฮไลท์ & ชื่อ",
       CurrentValue = false,
       Callback = function(Value) ESP_Active = Value SaveCurrentSettings() end,
    })

    local TPToggle = WarpTab:CreateToggle({
       Name = "วาร์ปเราไปหา (Auto-Return)",
       CurrentValue = false,
       Callback = function(Value) 
          TP_To_Item_Active = Value 
          if Value and LocalPlayer.Character then
              UserOriginalCF = LocalPlayer.Character.HumanoidRootPart.CFrame
          end
          SaveCurrentSettings()
       end,
    })

    local BringToggle = WarpTab:CreateToggle({
       Name = "วาร์ปวัตถุมา (Priority Orbit)",
       CurrentValue = false,
       Callback = function(Value) 
          Bring_To_Me_Active = Value 
          if not Value then
              for item, cf in pairs(OriginalItemCFrames) do if item and item.Parent then item.CFrame = cf end end
              table.clear(OriginalItemCFrames)
          end
          SaveCurrentSettings()
       end,
    })

    local BringRowToggle = WarpTab:CreateToggle({
        Name = "Bring (วาร์ปเรียงแถว 5xN - No Follow)",
        CurrentValue = false,
        Callback = function(Value) 
            Bring_Row_Active = Value 
            if not Value then
                for item, cf in pairs(OriginalItemCFrames) do if item and item.Parent then item.CFrame = cf end end
                table.clear(OriginalItemCFrames)
            end
            SaveCurrentSettings()
        end,
    })

    local HopToggle = WarpTab:CreateToggle({
       Name = "Auto Hop (ย้ายเซิร์ฟอัตโนมัติ)",
       CurrentValue = false,
       Callback = function(Value) Auto_Hop_Active = Value SaveCurrentSettings() end,
    })

    WarpTab:CreateButton({ Name = "Force Server Hop", Callback = DoServerHop })
    WarpTab:CreateButton({ Name = "Rejoin Server", Callback = Rejoin })

    local SaveToggle = SaveTab:CreateToggle({
       Name = "Auto Save Settings",
       CurrentValue = false,
       Callback = function(Value) 
           Auto_Save_Enabled = Value 
           SaveCurrentSettings() 
       end,
    })

    --- ### Load Data ###
    local savedData = LoadSavedSettings()
    if savedData then
        Auto_Save_Enabled = savedData.AutoSave
        SaveToggle:Set(savedData.AutoSave)
        if savedData.AutoSave then
            task.wait(0.5)
            ESPToggle:Set(savedData.ESP)
            TPToggle:Set(savedData.TP)
            BringToggle:Set(savedData.Orbit)
            BringRowToggle:Set(savedData.BringRow or false)
            HopToggle:Set(savedData.AutoHop)
        end
    end

    --- ### Main Runner Loop ###
    local function RunFarm()
        task.spawn(function()
            local orbitAngle = 0
            local followTarget = nil
            local lastBringPos = nil

            while true do
                local hrp = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
                local folder = workspace:FindFirstChild("BodyParts")
                local votedMap = workspace:FindFirstChild("VotedMap")
                local validItems = {}

                if folder and hrp then
                    for _, obj in pairs(folder:GetDescendants()) do
                        ManageClassicESP(obj)
                        if obj:IsA("BasePart") and obj.Transparency == 0 and table.find(TargetNames, obj.Name) then
                            if not (obj.Parent and obj.Parent:FindFirstChild(LocalPlayer.Name)) then
                                table.insert(validItems, obj)
                            end
                        end
                    end
                end

                -- Auto Hop Logic
                if Auto_Hop_Active then
                    local hasMap = (votedMap and #votedMap:GetChildren() > 0)
                    if (hasMap and #validItems == 0) or (not votedMap or #votedMap:GetChildren() == 0) then
                        task.wait(1)
                        DoServerHop()
                        break
                    end
                end

                -- Priority Orbit Logic (V22 Style)
                if Bring_To_Me_Active and hrp then
                    if #validItems == 0 then
                        if not Auto_Save_Enabled then BringToggle:Set(false) end
                    else
                        orbitAngle = orbitAngle + 0.1
                        table.sort(validItems, function(a, b)
                            local prio = {["Torso"] = 1, ["Head"] = 2}
                            return (prio[a.Name] or 99) < (prio[b.Name] or 99)
                        end)
                        for i, item in ipairs(validItems) do
                            if not OriginalItemCFrames[item] then OriginalItemCFrames[item] = item.CFrame end
                            item.CanCollide = false
                            if i == 1 then 
                                item.CFrame = hrp.CFrame * CFrame.new(0, math.sin(tick()*20)*0.5, 0)
                            else 
                                local x = math.cos(orbitAngle + (i * 0.5)) * 8
                                local z = math.sin(orbitAngle + (i * 0.5)) * 8
                                item.CFrame = hrp.CFrame * CFrame.new(x, 2, z)
                            end
                        end
                    end
                end

                -- Bring Row Logic (Instant Warp - No Follow)
                if Bring_Row_Active and hrp then
                    if #validItems > 0 then
                        if not lastBringPos then lastBringPos = hrp.CFrame end
                        for i, item in ipairs(validItems) do
                            if not OriginalItemCFrames[item] then OriginalItemCFrames[item] = item.CFrame end
                            item.CanCollide = false
                            item.Velocity = Vector3.new(0,0,0)
                            local col = ((i-1)%5-2)*4
                            local row = math.floor((i-1)/5)*4
                            item.CFrame = lastBringPos * CFrame.new(col, 0, -7-row)
                        end
                    end
                else
                    lastBringPos = nil
                end

                -- TP To Item Logic
                if TP_To_Item_Active and hrp then
                    if #validItems == 0 then
                        if not Auto_Save_Enabled then 
                            TPToggle:Set(false)
                            if UserOriginalCF then hrp.CFrame = UserOriginalCF UserOriginalCF = nil end
                        end
                    else
                        if not followTarget or not followTarget.Parent or followTarget.Transparency == 1 then
                            followTarget = validItems[math.random(1, #validItems)]
                        end
                        if followTarget then
                            local jitter = (tick() % 0.2 > 0.1) and 0.6 or -0.6
                            hrp.Velocity = Vector3.new(0,0,0)
                            hrp.CFrame = followTarget.CFrame * CFrame.new(0, 0, jitter)
                        end
                    end
                end
                task.wait(0.05)
            end
        end)
    end

    -- // AUTO-START EXECUTION // --
    task.wait(2)
    RunFarm()
]]

loadstring(scriptCode)()
