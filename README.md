-- ==========================================
-- TẢI THƯ VIỆN WINDUI & KHỞI TẠO CỬA SỔ
-- ==========================================
local WindUI = loadstring(game:HttpGet("https://github.com/Footagesus/WindUI/releases/latest/download/main.lua"))()

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local Workspace = game:GetService("Workspace")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local LocalPlayer = Players.LocalPlayer
local Camera = Workspace.CurrentCamera

-- Biến Trạng Thái Cục Bộ
local TargetPlayerName = ""
local BehindTargetEnabled, OrbitTargetEnabled = false, false
local KillAllEnabled, CameraViewEnabled, NoclipEnabled, AntiStunEnabled, InfiniteJumpEnabled = false, false, false, false, false

local TargetConnectionRender, OrbitLoopTask = nil, nil
local NoclipConnection, AntiStunConnection, InfJumpConnection = nil, nil, nil

local ESP_Enabled = false
local ESP_Color = Color3.fromRGB(160, 32, 240)

-- Kết nối Knit Service
local CombatService
task.spawn(function()
    pcall(function()
        local Knit = ReplicatedStorage:WaitForChild("Knit", 5)
        if Knit then
            local Services = Knit:WaitForChild("Services", 5)
            if Services then CombatService = Services:WaitForChild("CombatService", 5) end
        end
    end)
end)

-- ==========================================
-- HÀM XỬ LÝ KỸ THUẬT & HỖ TRỢ
-- ==========================================
local function GetPlayerList()
    local list = {}
    for _, plr in pairs(Players:GetPlayers()) do 
        if plr ~= LocalPlayer then 
            table.insert(list, plr.Name .. " (" .. plr.DisplayName .. ")") 
        end 
    end
    if #list == 0 then table.insert(list, "No player found") end
    return list
end

local function GetPlayerFromSelected(selectedString)
    if not selectedString or selectedString == "" or selectedString == "No player found" or selectedString == "Không tìm thấy người chơi" then return nil end
    for _, plr in pairs(Players:GetPlayers()) do
        if plr ~= LocalPlayer then
            local formattedName = plr.Name .. " (" .. plr.DisplayName .. ")"
            if selectedString == formattedName or selectedString == plr.Name or selectedString == plr.DisplayName then 
                return plr 
            end
        end
    end
    return nil
end

local function FireGodDamage(targetChar)
    if not targetChar then return end
    local head = targetChar:FindFirstChild("Head") or targetChar:FindFirstChild("HumanoidRootPart")
    if LocalPlayer.Character then
        local tool = LocalPlayer.Character:FindFirstChildOfClass("Tool")
        if tool then tool:Activate() end
    end
    if CombatService then
        local reFolder = CombatService:FindFirstChild("RE")
        if reFolder then
            for _, v in ipairs(reFolder:GetChildren()) do
                if v:IsA("RemoteEvent") then
                    pcall(function()
                        v:FireServer(targetChar)
                        if head then v:FireServer(targetChar, head) end
                    end)
                end
            end
        end
    end
end

local function StopLock()
    if TargetConnectionRender then TargetConnectionRender:Disconnect() TargetConnectionRender = nil end
    if OrbitLoopTask then task.cancel(OrbitLoopTask) OrbitLoopTask = nil end
    local localChar = LocalPlayer.Character
    if localChar then
        local humanoid = localChar:FindFirstChildOfClass("Humanoid")
        local hrp = localChar:FindFirstChild("HumanoidRootPart")
        if humanoid then humanoid.PlatformStand = false; pcall(function() humanoid:ChangeState(Enum.HumanoidStateType.GettingUp) end) end
        if hrp then hrp.Anchored = false end
    end
end

-- ==========================================
-- GIAO DIỆN CHÍNH
-- ==========================================
local Window = WindUI:CreateWindow({
    Title = "Gojo Hub v15.7",
    Icon = "rbxassetid://8064764164",
    Author = "Gojo Studio",
    Folder = "GojoHub_Config",
    Size = UDim2.fromOffset(580, 420),
    Transparent = true,
    Theme = "Dark",
})

local TabMain = Window:Tab({ Title = "Chính", Icon = "user" })
local TabCheater = Window:Tab({ Title = "Gian Lận", Icon = "zap" })
local TabESP = Window:Tab({ Title = "Nhìn Xuyên Tường", Icon = "eye" })
local TabCam = Window:Tab({ Title = "Theo Dõi", Icon = "camera" })
local TabMisc = Window:Tab({ Title = "Misc", Icon = "sliders" })
local TabDiscord = Window:Tab({ Title = "Discord", Icon = "message-square" })

-- ------------------------------------------
-- 1. TAB CHÍNH
-- ------------------------------------------
local TargetDropdown = TabMain:Dropdown({
    Title = "Chọn Mục Tiêu",
    Values = GetPlayerList(),
    Value = "",
    Callback = function(v)
        TargetPlayerName = v
    end
})

TabMain:Button({
    Title = "Làm Mới Danh Sách Người Chơi",
    Callback = function()
        TargetDropdown:SetValues(GetPlayerList())
    end
})

TabMain:Toggle({
    Title = "Bám Sau Lưng Mục Tiêu",
    Value = false,
    Callback = function(v)
        BehindTargetEnabled = v
        if not v then StopLock() return end
        
        TargetConnectionRender = RunService.RenderStepped:Connect(function()
            local targetPlr = GetPlayerFromSelected(TargetPlayerName)
            local myChar = LocalPlayer.Character
            if targetPlr and targetPlr.Character and myChar then
                local tHRP = targetPlr.Character:FindFirstChild("HumanoidRootPart")
                local myHRP = myChar:FindFirstChild("HumanoidRootPart")
                if tHRP and myHRP then
                    myHRP.CFrame = tHRP.CFrame * CFrame.new(0, 0, 3)
                end
            end
        end)
    end
})

TabMain:Toggle({
    Title = "Xoay Quanh Mục Tiêu",
    Value = false,
    Callback = function(v)
        OrbitTargetEnabled = v
        if not v then StopLock() return end
        
        local angle = 0
        OrbitLoopTask = task.spawn(function()
            while OrbitTargetEnabled do
                local targetPlr = GetPlayerFromSelected(TargetPlayerName)
                local myChar = LocalPlayer.Character
                if targetPlr and targetPlr.Character and myChar then
                    local tHRP = targetPlr.Character:FindFirstChild("HumanoidRootPart")
                    local myHRP = myChar:FindFirstChild("HumanoidRootPart")
                    if tHRP and myHRP then
                        angle = angle + 0.1
                        local offset = Vector3.new(math.cos(angle) * 5, 0, math.sin(angle) * 5)
                        myHRP.CFrame = CFrame.new(tHRP.Position + offset, tHRP.Position)
                    end
                end
                task.wait()
            end
        end)
    end
})

-- ------------------------------------------
-- 2. TAB GIAN LẬN
-- ------------------------------------------
TabCheater:Toggle({
    Title = "Nhảy Vô Tận",
    Value = false,
    Callback = function(v)
        InfiniteJumpEnabled = v
        if InfJumpConnection then InfJumpConnection:Disconnect() InfJumpConnection = nil end
        if InfiniteJumpEnabled then
            InfJumpConnection = UserInputService.JumpRequest:Connect(function()
                if InfiniteJumpEnabled and LocalPlayer.Character then
                    local hum = LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
                    if hum then
                        hum:ChangeState(Enum.HumanoidStateType.Jumping)
                    end
                end
            end)
        end
    end
})

TabCheater:Toggle({
    Title = "Chống Choáng Vẫn Di Chuyển Được",
    Value = false,
    Callback = function(v)
        AntiStunEnabled = v
        if AntiStunConnection then AntiStunConnection:Disconnect() AntiStunConnection = nil end
        if AntiStunEnabled then
            AntiStunConnection = RunService.Stepped:Connect(function()
                local char = LocalPlayer.Character
                if not char then return end
                
                local hum = char:FindFirstChildOfClass("Humanoid")
                if hum then
                    if hum.PlatformStand or hum:GetState() == Enum.HumanoidStateType.Ragdoll or hum:GetState() == Enum.HumanoidStateType.FallingDown then
                        hum.PlatformStand = false
                        pcall(function() hum:ChangeState(Enum.HumanoidStateType.GettingUp) end)
                    end
                    hum.AutoRotate = true
                end

                for _, child in ipairs(char:GetChildren()) do
                    local lowerName = string.lower(child.Name)
                    if string.find(lowerName, "stun") or string.find(lowerName, "freeze") or string.find(lowerName, "action") then
                        if child:IsA("BoolValue") or child:IsA("StringValue") then
                            child:Destroy()
                        end
                    end
                end

                for attr, val in pairs(char:GetAttributes()) do
                    local lowerAttr = string.lower(attr)
                    if string.find(lowerAttr, "stun") or string.find(lowerAttr, "disabled") or string.find(lowerAttr, "cantmove") then
                        char:SetAttribute(attr, false)
                    end
                end
            end)
        end
    end
})

TabCheater:Toggle({
    Title = "Tự Động Diệt Tất Cả",
    Value = false,
    Callback = function(v)
        KillAllEnabled = v
        if KillAllEnabled then
            task.spawn(function()
                while KillAllEnabled do
                    for _, targetPlr in ipairs(Players:GetPlayers()) do
                        if not KillAllEnabled then break end
                        if targetPlr ~= LocalPlayer and targetPlr.Character then
                            local myChar = LocalPlayer.Character
                            local myHRP = myChar and myChar:FindFirstChild("HumanoidRootPart")
                            local tHRP = targetPlr.Character:FindFirstChild("HumanoidRootPart")
                            local tHum = targetPlr.Character:FindFirstChildOfClass("Humanoid")
                            if myHRP and tHRP and tHum and tHum.Health > 0 then
                                myHRP.CFrame = tHRP.CFrame * CFrame.new(0, 0, 1.5)
                                FireGodDamage(targetPlr.Character)
                                task.wait(0.01)
                            end
                        end
                    end
                    task.wait(0.01)
                end
            end)
        end
    end
})

TabCheater:Toggle({
    Title = "Đi Xuyên Tường Vĩnh Viễn",
    Value = false,
    Callback = function(v)
        NoclipEnabled = v
        if NoclipConnection then NoclipConnection:Disconnect() NoclipConnection = nil end
        if NoclipEnabled then
            NoclipConnection = RunService.Stepped:Connect(function()
                if LocalPlayer.Character then
                    for _, p in pairs(LocalPlayer.Character:GetDescendants()) do
                        if p:IsA("BasePart") then p.CanCollide = false end
                    end
                end
            end)
        end
    end
})

-- TÍNH NĂNG MỚI: HACK TICK (GIẢ MÃ HOẶC TĂNG TỐC THỜI GIAN)
TabCheater:Toggle({
    Title = "Bật Giả Mạo Tick Thời Gian",
    Value = false,
    Callback = function(v)
        if v then
            local oldTick = tick
            local fakeTime = oldTick()
            pcall(function()
                hookfunction(tick, function(...)
                    fakeTime = fakeTime + 100
                    return fakeTime
                end)
            end)
            WindUI:Notify({ Title = "Gian Lận", Content = "Đã bật giả mạo tick thành công!", Duration = 2 })
        end
    end
})

-- TÍNH NĂNG MỚI: DASH VÔ HẠN
TabCheater:Toggle({
    Title = "Dash Vô Hạn",
    Value = false,
    Callback = function(v)
        if v then
            pcall(function()
                local rs = game:GetService("ReplicatedStorage")
                local plr = game.Players.LocalPlayer
                local movesetAttr = plr:GetAttribute("Moveset") or ""
                local serviceFolder = rs:FindFirstChild("Knit") and rs.Knit:FindFirstChild("Knit") and rs.Knit.Knit:FindFirstChild("Services")
                local targetService = serviceFolder and serviceFolder:FindFirstChild(movesetAttr .. "Service")
                local rem = targetService and targetService:FindFirstChild("RE") and targetService.RE:FindFirstChild("Activated")
                
                if rem then
                    local old
                    old = hookmetamethod(game, "__namecall", function(self, ...)
                        local m = getnamecallmethod()
                        local a = { ... }
                        if self == rem and m == "FireServer" then
                            if a[1] == false then
                                a[1] = "Down"
                            end
                            return old(self, unpack(a))
                        end
                        return old(self, ...)
                    end)
                    WindUI:Notify({ Title = "Gian Lận", Content = "Đã kích hoạt Dash Vô Hạn!", Duration = 2 })
                else
                    WindUI:Notify({ Title = "Lỗi", Content = "Không tìm thấy đường dẫn RemoteEvent phù hợp cho Dash!", Duration = 3 })
                end
            end)
        end
    end
})

-- ------------------------------------------
-- 3. TAB NHÌN XUYÊN TƯỜNG
-- ------------------------------------------
TabESP:Toggle({
    Title = "Khung Định Vị Người Chơi",
    Value = false,
    Callback = function(v)
        ESP_Enabled = v
        for _, plr in ipairs(Players:GetPlayers()) do
            if plr ~= LocalPlayer and plr.Character then
                local highlight = plr.Character:FindFirstChild("GojoESP")
                if v then
                    if not highlight then
                        highlight = Instance.new("Highlight")
                        highlight.Name = "GojoESP"
                        highlight.FillColor = ESP_Color
                        highlight.OutlineColor = Color3.fromRGB(255, 255, 255)
                        highlight.Parent = plr.Character
                    end
                else
                    if highlight then highlight:Destroy() end
                end
            end
        end
    end
})

TabESP:Colorpicker({
    Title = "Màu Khung Định Vị",
    Default = ESP_Color,
    Callback = function(color)
        ESP_Color = color
        for _, plr in ipairs(Players:GetPlayers()) do
            if plr.Character and plr.Character:FindFirstChild("GojoESP") then
                plr.Character.GojoESP.FillColor = color
            end
        end
    end
})

-- ------------------------------------------
-- 4. TAB THEO DÕI
-- ------------------------------------------
local CamDropdown = TabCam:Dropdown({
    Title = "Chọn Người Chơi",
    Values = GetPlayerList(),
    Value = "",
    Callback = function(v)
        if CameraViewEnabled then
            local targetPlr = GetPlayerFromSelected(v)
            if targetPlr and targetPlr.Character and targetPlr.Character:FindFirstChildOfClass("Humanoid") then
                Camera.CameraSubject = targetPlr.Character.Humanoid
            end
        end
    end
})

TabCam:Toggle({
    Title = "Bật Chế Độ Theo Dõi Màn Hình",
    Value = false,
    Callback = function(v)
        CameraViewEnabled = v
        if v then
            local targetPlr = GetPlayerFromSelected(CamDropdown.Value)
            if targetPlr and targetPlr.Character and targetPlr.Character:FindFirstChildOfClass("Humanoid") then
                Camera.CameraSubject = targetPlr.Character.Humanoid
            end
        else
            if LocalPlayer.Character and LocalPlayer.Character:FindFirstChildOfClass("Humanoid") then
                Camera.CameraSubject = LocalPlayer.Character.Humanoid
            end
        end
    end
})

-- ------------------------------------------
-- 5. TAB MISC (20 NGÔN NGỮ - MẶC ĐỊNH TIẾNG VIỆT)
-- ------------------------------------------
TabMisc:Dropdown({
    Title = "Language / Ngôn Ngữ",
    Values = {
        "Tiếng Việt 🇻🇳",
        "English 🇺🇸",
        "Español 🇪🇸",
        "Français 🇫🇷",
        "Deutsch 🇩🇪",
        "Português 🇧🇷",
        "Русский 🇷🇺",
        "日本語 🇯🇵",
        "한국어 🇰🇷",
        "中文 (简体) 🇨🇳",
        "中文 (繁體) 🇹🇼",
        "Bahasa Indonesia 🇮🇩",
        "ไทย 🇹🇭",
        "Tiếng Tagalog 🇵🇭",
        "Türkçe 🇹🇷",
        "العربية 🇸🇦",
        "Hindi 🇮🇳",
        "Italiano 🇮🇹",
        "Polski 🇵🇱",
        "Nederlands 🇳🇱"
    },
    Value = "Tiếng Việt 🇻🇳", -- Đã đặt Tiếng Việt làm mặc định
    Callback = function(v)
        WindUI:Notify({ Title = "Ngôn Ngữ / Language", Content = "Đã chọn / Selected: " .. v, Duration = 2 })
    end
})

TabMisc:Slider({
    Title = "Tốc Độ Di Chuyển (WalkSpeed)",
    Min = 16,
    Max = 200,
    Default = 16,
    Callback = function(v)
        if LocalPlayer.Character and LocalPlayer.Character:FindFirstChildOfClass("Humanoid") then
            LocalPlayer.Character.Humanoid.WalkSpeed = v
        end
    end
})

-- ------------------------------------------
-- 6. TAB DISCORD
-- ------------------------------------------
TabDiscord:Button({
    Title = "Sao Chép Liên Kết Máy Chủ Discord",
    Callback = function()
        setclipboard("https://discord.gg/gojostudio")
        WindUI:Notify({ Title = "Discord", Content = "Đã sao chép liên kết Discord thành công!", Duration = 3 })
    end
})

-- ==========================================
-- LẮNG NGHE SỰ KIỆN CẬP NHẬT NGƯỜI CHƠI
-- ==========================================
Players.PlayerAdded:Connect(function()
    task.wait(0.5)
    local list = GetPlayerList()
    TargetDropdown:SetValues(list)
    CamDropdown:SetValues(list)
end)

Players.PlayerRemoving:Connect(function()
    task.wait(0.5)
    local list = GetPlayerList()
    TargetDropdown:SetValues(list)
    CamDropdown:SetValues(list)
end)
