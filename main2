-- // AUTO-EXECUTE RELOADER \\ --
local script_to_run = [[loadstring(game:HttpGet("https://raw.githubusercontent.com/nigmaBoy/autofarm/refs/heads/main/main"))()]]
if syn and syn.queue_on_teleport then
    syn.queue_on_teleport(script_to_run)
elseif queue_on_teleport then
    queue_on_teleport(script_to_run)
end

if getgenv().OPTIMIZED_FARM_LOADED then return end
getgenv().OPTIMIZED_FARM_LOADED = true

local Rayfield = loadstring(game:HttpGet('https://sirius.menu/rayfield'))()
local Window = Rayfield:CreateWindow({Name = "OPTIMIZED FARM", LoadingTitle = "Loading...", ConfigurationSaving = {Enabled = false}})
local Tab = Window:CreateTab("Main", 4483362458)

local LP = game.Players.LocalPlayer
local RS = game:GetService("RunService")
local WS = workspace
local RE = game:GetService("ReplicatedStorage"):WaitForChild("MainEvent")
local TS = game:GetService("TeleportService")
local HttpService = game:GetService("HttpService")
local GuiService = game:GetService("GuiService")

-- // FILENAMES \\ --
local STATS_FILENAME = "OptimizedFarm_Stats.txt"
local HISTORY_FILENAME = "OptimizedFarm_History.txt"

-- // VARIABLES \\ --
local SafeCFrame = CFrame.new(-657.105, -45.438, -285.495)
local IdlePosition = SafeCFrame * CFrame.new(0, 20, 0)
local LockingPosition = IdlePosition
local IsLocking = true
local KOCount = 0 
local CameraLock = false

-- Persistent Stats
local totalMoneyGained = 0 
local totalTimeFarmed = 0  
local currentSessionMoney = 0 
local currentSessionTime = 0  
local lastExecutionTimestamp = 0

-- Live tracking
local serverStartMoney = 0
local serverStartTime = tick()

-- // UI LABELS \\ --
local SessionGainedLabel = Tab:CreateLabel("Session Gained: $0")
local MoneyPerMinLabel = Tab:CreateLabel("Session Money/Min: $0")
local MoneyPerHourLabel = Tab:CreateLabel("Session Money/Hour: $0")
Tab:CreateDivider()
local TotalGainedLabel = Tab:CreateLabel("Total Gained (All Time): $0")
local AvgMoneyPerHourLabel = Tab:CreateLabel("Average Money/Hour: $0")

-- // STATS SYSTEM \\ --
local function SaveStats()
    pcall(function()
        local data = {
            totalMoney = totalMoneyGained,
            totalTime = totalTimeFarmed,
            sessionMoney = currentSessionMoney,
            sessionTime = currentSessionTime,
            lastExec = tick()
        }
        writefile(STATS_FILENAME, HttpService:JSONEncode(data))
    end)
end

local function LoadStats()
    pcall(function()
        if isfile(STATS_FILENAME) then
            local data = HttpService:JSONDecode(readfile(STATS_FILENAME))
            totalMoneyGained = data.totalMoney or 0
            totalTimeFarmed = data.totalTime or 0
            if (tick() - (data.lastExec or 0)) > 3600 then
                currentSessionMoney = 0
                currentSessionTime = 0
            else
                currentSessionMoney = data.sessionMoney or 0
                currentSessionTime = data.sessionTime or 0
            end
            lastExecutionTimestamp = data.lastExec or tick()
        end
    end)
end

-- // SERVER HISTORY & HOPPING \\ --
local function GetServerHistory()
    local history = {}
    pcall(function()
        if isfile(HISTORY_FILENAME) then
            local data = HttpService:JSONDecode(readfile(HISTORY_FILENAME))
            for id, timestamp in pairs(data) do
                if (tick() - timestamp) < 600 then history[id] = timestamp end
            end
        end
    end)
    return history
end

local function AddToHistory(jobId)
    local history = GetServerHistory()
    history[jobId] = tick()
    writefile(HISTORY_FILENAME, HttpService:JSONEncode(history))
end

local function ServerHop()
    local serverElapsed = tick() - serverStartTime
    local currencyVal = (LP:FindFirstChild("DataFolder") and LP.DataFolder:FindFirstChild("Currency")) and LP.DataFolder.Currency.Value or serverStartMoney
    local serverGained = currencyVal - serverStartMoney
    
    if serverGained > 0 then
        totalMoneyGained = totalMoneyGained + serverGained
        totalTimeFarmed = totalTimeFarmed + serverElapsed
        currentSessionMoney = currentSessionMoney + serverGained
        currentSessionTime = currentSessionTime + serverElapsed
    end
    SaveStats()
    AddToHistory(game.JobId)

    local function TryTeleport()
        local url = "https://games.roblox.com/v1/games/" .. game.PlaceId .. "/servers/Public?sortOrder=Desc&limit=100"
        local success, result = pcall(function() return game:HttpGet(url) end)
        if not success then return end
        local data = HttpService:JSONDecode(result)
        local history = GetServerHistory()
        local validServers = {}
        for _, server in pairs(data.data) do
            if server.playing < server.maxPlayers and server.id ~= game.JobId and not history[server.id] then
                table.insert(validServers, server.id)
            end
        end
        if #validServers > 0 then
            TS:TeleportToPlaceInstance(game.PlaceId, validServers[math.random(1, #validServers)], LP)
        else
            TS:Teleport(game.PlaceId, LP)
        end
    end

    local errorConn; errorConn = GuiService.ErrorMessageChanged:Connect(function()
        task.wait(1)
        errorConn:Disconnect()
        ServerHop()
    end)

    TryTeleport()
    task.delay(15, function() if errorConn then errorConn:Disconnect() end TryTeleport() end)
end

-- // ANTI-KO & ANTI-GRAB MONITORING \\ --
local function CharacterMonitor(Char)
    if not Char then return end
    local BodyEffects = Char:WaitForChild("BodyEffects", 10)
    if BodyEffects then
        local KO = BodyEffects:WaitForChild("K.O", 5)
        local Grabbed = BodyEffects:WaitForChild("Grabbed", 5)

        if KO then
            KO:GetPropertyChangedSignal("Value"):Connect(function()
                if KO.Value == true then
                    KOCount = KOCount + 1
                    if KOCount >= 2 then ServerHop() end
                end
            end)
        end

        if Grabbed then
            Grabbed:GetPropertyChangedSignal("Value"):Connect(function()
                if Grabbed.Value == true then ServerHop() end
            end)
        end
    end
end

LP.CharacterAdded:Connect(CharacterMonitor)
if LP.Character then task.spawn(CharacterMonitor, LP.Character) end

-- // 1. INSTANT GRENADE WELD \\ --
WS.Ignored.ChildAdded:Connect(function(v)
    if v.Name == "Handle" or v.Name == "Grenade" then
        task.spawn(function()
            local door = WS.MAP.Map.Vault:FindFirstChild("VaultDoor")
            while v and v.Parent and door and door.Parent do
                v.CanCollide = false
                v.Velocity = Vector3.zero
                v.AssemblyLinearVelocity = Vector3.zero
                v.AssemblyAngularVelocity = Vector3.zero
                v.CFrame = door.CFrame
                RS.Heartbeat:Wait()
            end
        end)
    end
end)

-- // 2. PLAYER LOCKING & AIMBOT \\ --
RS.RenderStepped:Connect(function()
    if IsLocking and LockingPosition and LP.Character and LP.Character:FindFirstChild("HumanoidRootPart") then
        LP.Character.HumanoidRootPart.CFrame = LockingPosition
        LP.Character.HumanoidRootPart.Velocity = Vector3.zero
    end
    
    -- GRENADE AIMBOT
    if CameraLock then
        local door = WS.MAP.Map.Vault:FindFirstChild("VaultDoor")
        if door then
            workspace.CurrentCamera.CFrame = CFrame.lookAt(workspace.CurrentCamera.CFrame.Position, door.Position)
        end
    end
end)

-- // 3. GLOBAL AUTO PICKUP \\ --
task.spawn(function()
    while true do
        pcall(function()
            local Char = LP.Character
            if Char and Char:FindFirstChild("HumanoidRootPart") then
                for _, v in pairs(WS.Ignored.Drop:GetChildren()) do
                    if v.Name == "MoneyDrop" and v:FindFirstChild("ClickDetector") then
                        if (v.Position - Char.HumanoidRootPart.Position).Magnitude < 25 then 
                            fireclickdetector(v.ClickDetector)
                        end
                    end
                end
            end
        end)
        task.wait(0.2)
    end
end)

-- // 4. FUNCTIONS \\ --
local function GetTotalAmmo()
    local ammoFrame = LP.PlayerGui.MainScreenGui:FindFirstChild("AmmoFrame")
    if ammoFrame and ammoFrame:FindFirstChild("AmmoText") then
        local rawText = ammoFrame.AmmoText.Text
        return tonumber(string.match(rawText, "%d+")) or 0
    end
    return 0
end

local function Buy(name, duration)
    local Char = LP.Character
    if not Char or not Char:FindFirstChild("HumanoidRootPart") then return end
    Char.Humanoid:UnequipTools()
    local wasLocking = IsLocking
    IsLocking = false
    task.wait(0.1) 
    local s = WS.Ignored.Shop:FindFirstChild(name)
    if not s then if wasLocking then IsLocking = true end return end
    local p = s:FindFirstChild("Head") or s:FindFirstChild("Handle")
    local c = s:FindFirstChild("ClickDetector") or p:FindFirstChild("ClickDetector")
    if p and c then
        local old = Char.HumanoidRootPart.CFrame
        local t = tick()
        repeat
            LP.Character.HumanoidRootPart.CFrame = p.CFrame
            fireclickdetector(c)
            RS.Heartbeat:Wait()
        until tick() - t > (duration or 0.5)
        Char.HumanoidRootPart.CFrame = old
    end
    if wasLocking then IsLocking = true end
end

local function FireBurst(vault, Gun)
    local Handle = Gun:FindFirstChild("Handle")
    local TargetPart = vault:FindFirstChild("Head") or vault:FindFirstChild("Part")
    if not TargetPart or not Handle then return end
    local gunRemote = Gun:FindFirstChild("RemoteEvent")
    if gunRemote then gunRemote:FireServer("Shoot") end
    local MuzzlePos = (Handle.CFrame * CFrame.new(-0.1, 0.5, -2.5)).Position 
    local TimeNow = workspace:GetServerTimeNow() 
    for i = 1, 5 do
        local v17 = math.random() > 0.5 and math.random() * 0.05 or -math.random() * 0.05
        local v18 = math.random() > 0.5 and math.random() * 0.1 or -math.random() * 0.1
        local v19 = math.random() > 0.5 and math.random() * 0.05 or -math.random() * 0.05
        local EndPos = TargetPart.Position + Vector3.new(v17, v18, v19)
        RE:FireServer("ShootGun", Handle, MuzzlePos, EndPos, TargetPart, Vector3.new(0, 1, 0), TimeNow)
    end
end

local function ThrowGrenadePair()
    local Char = LP.Character
    if not Char then return end
    for _, item in ipairs(LP.Backpack:GetChildren()) do
        if item.Name == "[Grenade]" then item:Destroy() end
    end
    Buy("[Grenade] - $788", 0.8)
    task.wait(0.2)
    for i = 1, 2 do
        local grenade = LP.Backpack:FindFirstChild("[Grenade]")
        if grenade then
            Char.Humanoid:EquipTool(grenade)
            task.wait(0.4)
            if grenade.Parent == Char then
                 grenade:Activate(); task.wait(0.2); grenade:Activate(); task.wait(0.5)
            else break end
        else break end
    end
    Char.Humanoid:UnequipTools()
end

-- // 5. FARM LOGIC \\ --
local function FarmVaults()
    local cashiers = WS:FindFirstChild("Cashiers")
    if not cashiers then return end
    local Gun = LP.Backpack:FindFirstChild("[Drum-Shotgun]") or LP.Character:FindFirstChild("[Drum-Shotgun]")
    if not Gun then return end
    Gun.Parent = LP.Character
    local killableVaults = {}
    for _, v in pairs(cashiers:GetChildren()) do
        if v.Name == "VAULT" then
            local Humanoid = v:FindFirstChild("Humanoid")
            if Humanoid and Humanoid.Health > 0 then table.insert(killableVaults, v) end
        end
    end
    if #killableVaults == 0 then ServerHop() return end
    for i, currentVault in ipairs(killableVaults) do
        local Humanoid = currentVault:FindFirstChild("Humanoid")
        local Head = currentVault:FindFirstChild("Head")
        if Head and Humanoid and Humanoid.Health > 0 then
            LockingPosition = Head.CFrame * CFrame.new(0, 0, 4.5) * CFrame.Angles(0, math.pi, 0)
            IsLocking = true
            while currentVault and Humanoid.Health > 0 and Gun.Parent == LP.Character do
                local MagAmmo = Gun:FindFirstChild("Ammo")
                if MagAmmo and MagAmmo.Value > 0 then
                    FireBurst(currentVault, Gun)
                    task.wait(0.425) 
                elseif MagAmmo then
                    RE:FireServer("Reload", Gun)
                    task.wait(1.5)
                end
            end
            if Gun then RE:FireServer("Reload", Gun) end
            local AnyMoneyLeft = true
            local Timeout = tick()
            while AnyMoneyLeft do
                AnyMoneyLeft = false 
                if LP.Character and LP.Character:FindFirstChild("HumanoidRootPart") then
                    for _, drop in pairs(WS.Ignored.Drop:GetChildren()) do
                        if drop.Name == "MoneyDrop" and drop:FindFirstChild("ClickDetector") then
                            if (drop.Position - LP.Character.HumanoidRootPart.Position).Magnitude < 15 then
                                fireclickdetector(drop.ClickDetector)
                                AnyMoneyLeft = true 
                            end
                        end
                    end
                end
                if AnyMoneyLeft then task.wait(0.15) end
                if tick() - Timeout > 6 then break end
            end
        end
    end
    LockingPosition = IdlePosition
    IsLocking = true
    ServerHop()
end

-- // MAIN RUNNER \\ --
local function RunFarm()
    local loadStart = tick()
    while not LP.Character or not LP.Character:FindFirstChild("FULLY_LOADED_CHAR") do
        if tick() - loadStart > 15 then ServerHop() return end
        task.wait(0.5)
    end
    serverStartMoney = LP:WaitForChild("DataFolder"):WaitForChild("Currency").Value
    serverStartTime = tick()
    
    IsLocking = true
    LockingPosition = IdlePosition

    local Gun = LP.Backpack:FindFirstChild("[Drum-Shotgun]") or LP.Character:FindFirstChild("[Drum-Shotgun]")
    if not Gun then
        Buy("[Drum-Shotgun] - $1238", 1.0)
        local t = tick()
        repeat task.wait() until LP.Backpack:FindFirstChild("[Drum-Shotgun]") or tick() - t > 3
        Gun = LP.Backpack:FindFirstChild("[Drum-Shotgun]")
    end
    if Gun then
        Gun.Parent = LP.Character
        if GetTotalAmmo() < 25 then Buy("18 [Drum-Shotgun Ammo] - $73", 1.3) end
        Gun = LP.Backpack:FindFirstChild("[Drum-Shotgun]") or LP.Character:FindFirstChild("[Drum-Shotgun]")
        if Gun then Gun.Parent = LP.Character end
        
        local door = WS.MAP.Map.Vault:FindFirstChild("VaultDoor")
        if door then
            -- ENTER FIRST PERSON AND AIMBOT ON
            LP.CameraMode = Enum.CameraMode.LockFirstPerson
            CameraLock = true
            
            local doorBreakTimeout = tick()
            while WS.MAP.Map.Vault:FindFirstChild("VaultDoor") do
                ThrowGrenadePair()
                local waitStart = tick()
                repeat task.wait(0.25) until not WS.MAP.Map.Vault:FindFirstChild("VaultDoor") or tick() - waitStart > 5
                if tick() - doorBreakTimeout > 25 then break end
            end
            
            -- EXIT FIRST PERSON AND AIMBOT OFF
            CameraLock = false
            LP.CameraMode = Enum.CameraMode.Classic
        end
        
        if not WS.MAP.Map.Vault:FindFirstChild("VaultDoor") then 
            FarmVaults() 
        else 
            ServerHop() 
        end
    else
        ServerHop()
    end
end

-- // UI UPDATER LOOP \\ --
task.spawn(function()
    while task.wait(1) do
        pcall(function()
            local serverElapsed = tick() - serverStartTime
            local serverGained = LP.DataFolder.Currency.Value - serverStartMoney
            local totalSessionMoney = currentSessionMoney + serverGained
            local totalSessionTime = currentSessionTime + serverElapsed
            if totalSessionTime > 0 then
                local mPM = (totalSessionMoney / totalSessionTime) * 60
                local mPH = mPM * 60
                SessionGainedLabel:Set("Session Gained: $"..math.floor(totalSessionMoney))
                MoneyPerMinLabel:Set("Session Money/Min: $"..math.floor(mPM))
                MoneyPerHourLabel:Set("Session Money/Hour: $"..math.floor(mPH))
            end
            TotalGainedLabel:Set("Total Gained (All Time): $"..math.floor(totalMoneyGained + serverGained))
            local allTimeF = totalTimeFarmed + serverElapsed
            if allTimeF > 0 then
                local avgPH = ((totalMoneyGained + serverGained) / allTimeF) * 3600
                AvgMoneyPerHourLabel:Set("Average Money/Hour: $"..math.floor(avgPH))
            end
            SaveStats()
        end)
    end
end)

Tab:CreateButton({ Name = "Start Farm Manually", Callback = RunFarm })
Tab:CreateButton({ Name = "Force Server Hop", Callback = ServerHop })

LoadStats()
task.wait(2)
RunFarm()
