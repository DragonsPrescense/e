local Whitelist = {no}

if SCLOADED then
    game:GetService("StarterGui"):SetCore("SendNotification", {
    	Title = "Error!",
    	Text = "Script Already Loaded!",
    	Icon = "rbxthumb://type=Asset&id=174811607&w=420&h=420",
    	Duration = 5,
    })
    return
end

pcall(function() getgenv().SCLOADED  = true end)

local function change(v)
    if v.Name == "HumanoidRootPart" and v.Parent.Name ~= "Log" then
        v.Parent.Parent = workspace.Mobs
    end
end
for _,moby in pairs(workspace.Mobs:GetDescendants()) do
    change(moby)
end
workspace.Mobs.DescendantAdded:Connect(function(v)
    change(v)
end)


------------------------------------------------------------------


local Players = game:GetService("Players")
if not Players.LocalPlayer then
    Players:GetPropertyChangedSignal("LocalPlayer"):Wait()
end
local Player = Players.LocalPlayer
repeat wait() until game:GetService("ReplicatedStorage"):FindFirstChild("HandlerFire")
local MainRemote = game:GetService("ReplicatedStorage").HandlerFire
local PlayerValue = game.ReplicatedStorage.PlayerValues
local SetUpUsers = {} -- table to keep track of users who have the chatted MainRemote hooked up to them
local F3xEnabled = false
local FakePart = Instance.new("Part")
local meta = getrawmetatable(game)
local newindex
local namecall


newindex = hookfunction(meta.__newindex, function(self, k, new)
    if tostring(string.lower(k)) == "cframe" and tostring(self.ClassName):find("Part") and checkcaller() then
        MainRemote.FireServer(MainRemote,"Module","SetCFrame",self, new)
    end
    return newindex(self,k,new)
end)

namecall = hookmetamethod(game, "__namecall", function(Self, ...)
    if Self.IsA(Self, "BasePart") and checkcaller() and string.lower(getnamecallmethod()) == "destroy" then
        MainRemote.FireServer(MainRemote,"Module","SetCFrame",Self, CFrame.new(0,game.Workspace.FallenPartsDestroyHeight,0))
        local FakePartToDestroy = Instance.new("Part")
        return namecall(FakePartToDestroy, ...)
    end
    return namecall(Self, ...)
end)

local function GrabTargets(name, speaker)
    if speaker ~= nil then
        if name == "me" or name == "myself" then
            return speaker
        end
    end
    local PlayerList = Players:GetPlayers()
    name = string.lower(name)
    if name == "others" or name == "all" or name == "random" then
        if name == "others" then
            for i = 1,#PlayerList do 
                if PlayerList[i] == Player or PlayerList[i] == speaker then
                    table.remove(PlayerList, i)   
                end
            end
        end
        if name == "others" or name == "all" then
            return PlayerList
        else
            return PlayerList[math.random(1,#PlayerList)]
        end
    end
    
    local UserMatch = nil
    local DisplayMatch = nil

    for i,v in pairs(PlayerList) do
        local UserInput = string.lower(v.Name)
        local DisplayInput = string.lower(v.DisplayName)                                     
        if string.sub(UserInput, 1, #name) == name then 
            UserMatch = v
        end
        if string.sub(DisplayInput, 1, #name) == name then 
            DisplayMatch = v
        end
    end
    if UserMatch ~= nil then 
        return UserMatch
    else
        return DisplayMatch
    end     
    print("SOMETHING WENT FUCKY WUCKY, HOW DID THIS HAPPEN WTH") -- this should never print LOL
end

-- Returns either a table, or a single user depending on how it's fired.
-- Others returns everyone other than the localplayer, all returns everyone
-- random returns a random user, and anything else will search for a username
-- In that case, it'll return one player, and prioritize regular names. If no matches to that exists, returns display name matches
-- (returns nil if there's no matches)

local function getargs(inputstring)
    inputstring = string.lower(inputstring)
    inputstring = string.gsub(inputstring, "'", "")
    inputstring = string.gsub(inputstring, ":", "")
    inputstring = string.gsub(inputstring, ";", "")
    inputstring = string.gsub(inputstring, "%.", "")
    inputstring = string.gsub(inputstring, "/e ", "")
    inputstring = string.gsub(inputstring, "/w ", "")
    local args = string.split(inputstring, " ")
    args.Command = args[1]
    table.remove(args, 1)
    return args
end

local function IsInTable(table, tofind)
    local found = false
    for i,v in pairs(table) do
        if v == tofind then
            found = true
            break
        end
    end
    return found
end

local function check(a)
    if a ~= nil and a.Character and a.Character:FindFirstChild("HumanoidRootPart") and a.Character:FindFirstChild("Humanoid") and a.Character.Humanoid.Health ~= 0 then
         return true
    end
end

local function kill(a)
    if a.Parent == Players then
        if check(a) then
            MainRemote:FireServer("Module","SetCFrame",a.Character.HumanoidRootPart,CFrame.new(0,workspace.FallenPartsDestroyHeight,0))
        end
    else
        if a:FindFirstChild("HumanoidRootPart") and a:FindFirstChild("Humanoid") and a.Humanoid.Health ~= 0 then
            if a.Humanoid.MaxHealth > 400 then
                repeat wait()
                    if a:FindFirstChild("HumanoidRootPart") and a:FindFirstChild("Humanoid") and a.Humanoid.Health ~= 0 then
                        game:GetService("ReplicatedStorage").Remotes.DamageHandler:FireServer(a.Humanoid)
                        MainRemote:FireServer("Module", "TakeDamage", a.Humanoid, 400)
                    end
                until a.Humanoid.Health == 0
            else
                if a:FindFirstChild("HumanoidRootPart") and a:FindFirstChild("Humanoid") and a.Humanoid.Health ~= 0 then
                    game:GetService("ReplicatedStorage").Remotes.DamageHandler:FireServer(a.Humanoid)
                    MainRemote:FireServer("Module", "TakeDamage", a.Humanoid, 400)
                end
            end
        end
    end
end

local function void(a)
    if check(a) then
        MainRemote:FireServer("Module","SetCFrame",a.Character.HumanoidRootPart,CFrame.new(0,99999999999999999,0))
    end
end

local function tp(being,to)
    if check(being) and check(to) then
        MainRemote:FireServer("Module","SetCFrame",being.Character.HumanoidRootPart,to.Character.HumanoidRootPart.CFrame + to.Character.HumanoidRootPart.CFrame.lookVector * 3)
    end
end


local function swap(being,to)
    if check(being) and check(to) then
        local old = being.Character.HumanoidRootPart.CFrame
        local old2 = to.Character.HumanoidRootPart.CFrame
        MainRemote:FireServer("Module","SetCFrame",being.Character.HumanoidRootPart,old2)
        MainRemote:FireServer("Module","SetCFrame",to.Character.HumanoidRootPart,old)
    end
end

local function punish(a)
    if check(a) then
        local distance = Vector3.new(99999999999999999,99999999999999999,99999999999999999)
        MainRemote:FireServer("Module", "Knockback", a, a.Character.HumanoidRootPart, distance, distance)
    end
end

local function god(a)
    if check(a) and a.Character.Humanoid.Health < 1000000000 then
        for i,v in pairs(game.Workspace.Zones:GetChildren()) do
            if v.Name == "Part" then
                local old = v.CFrame
                MainRemote:FireServer("Module", "SetCFrame", v, CFrame.new(0,10000000,0))
                game:GetService("ReplicatedStorage").Remotes.DamageHandler:FireServer(a.Character.Humanoid)
                MainRemote:FireServer("Module", "TakeDamage", a.Character.Humanoid, -math.huge)
                MainRemote:FireServer("Module", "SetCFrame", v, old)
            end
        end
    end
end

local function heal(a)
    if check(a) then
        game:GetService("ReplicatedStorage").Remotes.DamageHandler:FireServer(a.Character.Humanoid)
        MainRemote:FireServer("Module", "TakeDamage", a.Character.Humanoid, a.Character.Humanoid.Health - a.Character.Humanoid.MaxHealth)
    end
end

local function freeze(a)
    if check(a) and PlayerValue:FindFirstChild(a.Name) then
        game:GetService("ReplicatedStorage").HandlerFire:FireServer("Module", "CreateObject", nil, PlayerValue[a.Name], "BoolValue", nil, {["Name"] = "Stun"})
    end
end

local function unfreeze(a)
    if check(a) and PlayerValue:FindFirstChild(a.Name) and PlayerValue[a.Name]:FindFirstChild("Stun") then
        game:GetService("ReplicatedStorage").HandlerFire:FireServer("Module", "Remove",PlayerValue[a.Name].Stun)
    end
end

local function rocks()
    local Walls = {{Vector3.new(243.05000305176, 315, 156.25),Enum.Material.Slate,CFrame.new(1642.99756, 2237.36304, 433.854675, 0.123249374, 0.332972705, 0.934846938, -0.0711630881, 0.942569494, -0.326341212, -0.989820957, -0.0263052583, 0.139866471)},{Vector3.new(333.0153503418, 222, 156.96156311035),Enum.Material.Slate,CFrame.new(1616.14026, 2180.19775, 601.659973, -0.04490868, 0.134199291, 0.989936233, -0.0575037301, 0.988945663, -0.136673674, -0.997334719, -0.0630628467, -0.0366952866)},{Vector3.new(190.05000305176, 380, 386.96154785156),Enum.Material.Slate,CFrame.new(1865.47705, 2121.24756, 796.775513, -0.0981196538, 0.995159328, -0.00551150739, 0.0262800641, 0.00812735409, 0.99962157, 0.994827569, 0.0979376808, -0.0269503053)},{Vector3.new(192.84999084473, 191, 197.25),Enum.Material.Slate,CFrame.new(2141.78296, 2175.13159, 628.425049, -0.000493491709, -0.294381768, -0.955687761, -0.000281349872, 0.955687821, -0.294381648, 0.999999821, 0.000123607577, -0.000554448285)},{Vector3.new(277.04998779297, 308, 172.05000305176),Enum.Material.Slate,CFrame.new(2153.28833, 2111.46362, 406.231903, -0.0903602988, -0.172889799, -0.980787516, -0.052151531, 0.984286487, -0.168701842, 0.994542718, 0.0359056592, -0.0979568958)},{Vector3.new(574.01538085938, 462, 199.05000305176),Enum.Material.Slate,CFrame.new(1804.33813, 2192.68408, 266.66391, 0.984190345, 0.0188479908, -0.176108137, -0.0458799303, 0.987512112, -0.15071404, 0.171068266, 0.156411126, 0.97276473)},{Vector3.new(243.05000305176, 315, 156.25),Enum.Material.Slate,CFrame.new(1642.99756, 2237.36304, 433.854675, 0.123249374, 0.332972705, 0.934846938, -0.0711630881, 0.942569494, -0.326341212, -0.989820957, -0.0263052583, 0.139866471)},}
    for i,v in pairs(workspace.Map:GetChildren()) do
        if v:IsA("BasePart") then
            for x = 1,#Walls do
                if v.Size == Walls[x][1] and v.Material == Walls[x][2] then
                    MainRemote:FireServer("Module","SetCFrame",v,Walls[x][3])
                end
            end
        end
    end
end

local function save()
    for i,v in pairs(workspace.Mobs:GetChildren()) do
        if v:FindFirstChild("Humanoid") and v.Humanoid.Health ~= 0 then
            mob = v
        end
    end
    game:GetService("ReplicatedStorage").Remotes.DamageHandler:FireServer(mob.Humanoid)
    game:GetService("ReplicatedStorage").HandlerFire:FireServer("Module", "TakeDamage", mob.Humanoid, 400)
end

local function togglef3x(value)
    local f3x = Player.Backpack:FindFirstChild("F3X") or Player.Character:FindFirstChild("F3X")
    if f3x == nil then
        game:GetService("StarterGui"):SetCore("SendNotification", {
            Title = "LOADING F3X",
            Text = "This takes a while the first time (and each time you die)",
            Icon = "rbxassetid://2541869220",
            Duration = 4,
        })
        loadstring(game:HttpGet(('https://raw.githubusercontent.com/Aidez/MISC/main/f3xwithonlymoveandrotate'),true))()
    end
    if value ~= nil then
        F3xEnabled = value
    else 
        F3xEnabled = not F3xEnabled
    end
    if F3xEnabled == false then
        if Player.Character ~= nil and Player.Character:FindFirstChild("F3X") then
            game.Players.LocalPlayer.Character:FindFirstChild("F3X").Parent = Player.Backpack
        end
        return
    end
    repeat wait() until Player.Character ~= nil
    repeat wait() until Player.Character:FindFirstChild("F3X")
    repeat wait() until Player.PlayerGui:FindFirstChild("Building Tools by F3X (UI)")
    local WhitelistedImageNames = {"DeleteButton","UndoButton","RedoButton","MoveButton","RotateButton"}
    for i,v in pairs(Player.PlayerGui:FindFirstChild("Building Tools by F3X (UI)"):GetDescendants()) do
        if not table.find(WhitelistedImageNames, v.Name) and v:IsA("ImageButton") then
            v.Visible = false
        end
    end
end

local aura,killing,voiding,godded,teleporting,oldcool,movey = {},{},{},{},{},{},{},{}

local function CommandHandler(msg, speaker) -- function to attach you, and admins Chatted MainRemote to
    if not IsInTable(Whitelist, tostring(speaker)) and speaker ~= Player then
        return
    end 

    if string.sub(msg,0,1) ~=  "." and string.sub(msg,0,1) ~=  ":" and string.sub(msg,0,1) ~=  ";" and not string.find(msg, "/e ") and not string.find(msg, "/w ") then
        return
    end

    local Args = getargs(msg) 

    if IsInTable({"whitelist","wl","admin","givecommands","rank"}, Args.Command) and speaker == Player then
        local Target = GrabTargets(tostring(Args[1]))
        if Target == nil then
            return
        end
        if Target == Player then
            return
        end
        if type(Target) == "table" then
            for i,v in pairs(Target) do
                if not IsInTable(Whitelist, v.Name) and v ~= Player then
                    table.insert(Whitelist, v.Name)
                    if Player.PlayerGui:FindFirstChild("Leaderboard") then
                        for x,y in pairs(Player.PlayerGui.Leaderboard:GetDescendants()) do
                            if y.Name == "name" and y:FindFirstAncestor(v.Name) then
                                y.TextColor3 = Color3.new(1, 170/255, 0)
                            end
                        end
                    end
                    if not IsInTable(SetUpUsers, v.Name) then
                        SetUpUsers[v.Name] = true
                        v.Chatted:Connect(function(msg)
                            CommandHandler(msg, v)
                        end)
                        game:GetService("StarterGui"):SetCore("SendNotification", {
                            Title = "Whitelist!",
                            Text = v.Name.." Has Been Whitelisted!",
                            Icon = "rbxthumb://type=Asset&id=2022095751&w=420&h=420",
                            Duration = 5,
                        })
                        game:GetService("ReplicatedStorage").DefaultChatSystemChatEvents.SayMessageRequest:FireServer("/w "..v.Name.." You Have Been Whitelisted Say :cmds To View The Commands!", "All")
                    end
                else
                    game:GetService("StarterGui"):SetCore("SendNotification", {
                        Title = "Whitelist!",
                        Text = v.Name.." Is Already Whitelisted!",
                        Icon = "rbxthumb://type=Asset&id=174811607&w=420&h=420",
                        Duration = 5,
                    })
                end
            end
        else
            if not IsInTable(Whitelist, Target.Name) then
                table.insert(Whitelist, Target.Name) 
                if Player.PlayerGui:FindFirstChild("Leaderboard") then
                    for i,v in pairs(Player.PlayerGui.Leaderboard:GetDescendants()) do
                        if v.Name == "name" and v:FindFirstAncestor(Target.Name) then
                            v.TextColor3 = Color3.new(1, 170/255, 0)
                        end
                    end
                end
                if not IsInTable(SetUpUsers, Target.Name) then
                    SetUpUsers[Target.Name] = true
                    Target.Chatted:Connect(function(msg)
                        CommandHandler(msg, Target)
                    end)
                    game:GetService("StarterGui"):SetCore("SendNotification", {
                        Title = "Whitelist!",
                        Text = Target.Name.." Has Been Whitelisted!",
                        Icon = "rbxthumb://type=Asset&id=2022095751&w=420&h=420",
                        Duration = 5,
                    })
                    game:GetService("ReplicatedStorage").DefaultChatSystemChatEvents.SayMessageRequest:FireServer("/w "..Target.Name.." You Have Been Whitelisted Say :cmds To View The Commands!", "All")
                end
            else
                game:GetService("StarterGui"):SetCore("SendNotification", {
                    Title = "Whitelist!",
                    Text = Target.Name.." Is Already Whitelisted!",
                    Icon = "rbxthumb://type=Asset&id=174811607&w=420&h=420",
                    Duration = 5,
                })
            end
        end
    elseif IsInTable({"unwhitelist","unwl","uwl","unadmin","ungivecommands","takecommands","unrank"}, Args.Command) and speaker == Player then
        if Args[1] == "others" or Args[1] == "all" then
            for i = 1,#Whitelist do
                table.remove(Whitelist, 1)
            end
            if Player.PlayerGui:FindFirstChild("Leaderboard") then
                for i,v in pairs(Player.PlayerGui.Leaderboard:GetDescendants()) do
                    if v.Name == "name" then
                        v.TextColor3 = Color3.new(0, 0, 0)
                    end
                end
            end
            return
        end
        for i,v in pairs(Whitelist) do
            if string.lower(string.sub(v, 0,#Args[1])) == string.lower(Args[1]) then
                table.remove(Whitelist, i)
                game:GetService("StarterGui"):SetCore("SendNotification", {
                    Title = "Rank!",
                    Text = v.." Has Been Unwhitelisted!",
                    Icon = "rbxthumb://type=Asset&id=2022095751&w=420&h=420",
                    Duration = 5,
                })
                if Player.PlayerGui:FindFirstChild("Leaderboard") then
                    for x,y in pairs(Player.PlayerGui.Leaderboard:GetDescendants()) do
                        if y.Name == "name" and y:FindFirstAncestor(v) then
                            y.TextColor3 = Color3.new(0, 0, 0)
                        end
                    end
                end
            end
        end
    elseif IsInTable({"wls","printwhitelists","whitelists","printwl","wllist","printwhitelist","admins","moderators","mods"}, Args.Command) and speaker == Player then
        print("--- Printing Whitelist List ---")
        for i,v in pairs(Whitelist) do
            print(i..": "..v)
        end
        print("------------- Done ------------")
    elseif IsInTable({"commands","cmds","cmdslist"}, Args.Command) and speaker ~= Player and Args[1] == nil then
        game:GetService("ReplicatedStorage").DefaultChatSystemChatEvents.SayMessageRequest:FireServer("/w "..speaker.Name.." :kill, :heal, :freeze, :bring, :goto, :fling, :god, :1v1, :loopkill, :loopfling, :loopbring, :aura", "All")
    elseif IsInTable({"loopkill","lk","loop","infkill"}, Args.Command) then
        local Target = GrabTargets(tostring(Args[1]))
        if Target == nil then
            return
        end
        if Target == Player then
            return
        end
        if type(Target) == "table" then
            for i,v in pairs(Target) do
                if not IsInTable(killing, v.Name) and v ~= Player then
                    table.insert(killing, v.Name)
                end
            end
        else
            if not IsInTable(killing, Target.Name) then
                table.insert(killing, Target.Name)
            end
        end
    elseif IsInTable({"unloopkill","unlk","ulk","unloop","uninfkill"}, Args.Command)  then
        if Args[1] == "others" or Args[1] == "all" then
            for i = 1,#killing do
                table.remove(killing, 1)
            end
            return
        end
        for i,v in pairs(killing) do
            if string.lower(string.sub(v, 0,#Args[1])) == string.lower(Args[1]) then
                table.remove(killing, i)
            end
        end
    elseif IsInTable({"kill","murder","erase,","dump","dunkon","eliminate","end","destroy"}, Args.Command) then
        if Args[1] == "me" then
            return
        end
        if Args[1] == "mobs" then
            for _,mob in pairs(workspace.Mobs:GetChildren()) do
                if mob:FindFirstChild("Humanoid") and mob.Humanoid.Health ~= 0 then
                    kill(mob)
                end
            end
            for _,mob2 in pairs(workspace.NPCS:GetChildren()) do
                if mob2:FindFirstChild("Humanoid") and mob2.Humanoid.Health ~= 0 then
                    kill(mob2)
                end
            end
            if workspace:FindFirstChild("Cocoons") then
                for _,mob3 in pairs(workspace.Cocoons:GetChildren()) do
                    if mob3:FindFirstChild("Humanoid") and mob3.Humanoid.Health ~= 0 then
                        kill(mob3)
                        wait(0.7)
                    end
                end
            end
        end
        for _,mob in pairs(workspace.Mobs:GetChildren()) do
            if string.lower(string.sub(mob.Name,0,#Args[1])) == string.lower(Args[1]) then
                kill(mob)
            end
        end
        for _,mob2 in pairs(workspace.NPCS:GetChildren()) do
            if string.lower(string.sub(mob2.Name,0,#Args[1])) == string.lower(Args[1]) then
                kill(mob2)
            end
        end
        if workspace:FindFirstChild("Cocoons") then
            for _,mob3 in pairs(workspace.Cocoons:GetChildren()) do
                if string.lower(string.sub(mob3.Name,0,#Args[1])) == string.lower(Args[1]) then
                    kill(mob3)
                end
            end
        end
        local Target = GrabTargets(tostring(Args[1]), speaker)
        if Target == nil then
            return
        end
        if Target == Player then
            return
        end
        if type(Target) == "table" then
            for i,v in pairs(Target) do
                kill(v)
            end
        else
            kill(Target)
        end
    elseif IsInTable({"heal","hp","help","givehp","full"}, Args.Command) then
        local Target = GrabTargets(tostring(Args[1]), speaker)
        if Target == nil then
            return
        end
        if type(Target) == "table" then
            for i,v in pairs(Target) do
                heal(v)
            end
        else
            heal(Target)
        end
    elseif IsInTable({"void","fling","space","sky"}, Args.Command) then
        local Target = GrabTargets(tostring(Args[1]), speaker)
        if Target == nil then
            return
        end
        if Target == Player then
            return
        end
        if type(Target) == "table" then
            for i,v in pairs(Target) do
                void(v)
            end
        else
            void(Target)
        end
    elseif IsInTable({"loopvoid","loopfling","loopspace","lv","lf","ls"}, Args.Command) then
        local Target = GrabTargets(tostring(Args[1]), speaker)
        if Target == nil then
            return
        end
        if Target == Player then
            return
        end
        if type(Target) == "table" then
            for i,v in pairs(Target) do
                if not table.find(voiding,v.Name) then
                    table.insert(voiding,v.Name)
                end
            end
        else
            if not table.find(voiding,Target.Name) then
                table.insert(voiding,Target.Name)
            end
        end
    elseif IsInTable({"unloopvoid","unloopfling","unloopspace","unlv","unlf","unls"}, Args.Command) then
        local Target = GrabTargets(tostring(Args[1]), speaker)
        if Target == nil then
            return
        end
        if type(Target) == "table" then
            for i,v in pairs(Target) do
                if table.find(voiding,v.Name) then
                    table.remove(voiding,table.find(voiding,v.Name))
                end
            end
        else
            if table.find(voiding,Target.Name) then
                table.remove(voiding,table.find(voiding,Target.Name))
            end
        end
    elseif IsInTable({"tp","teleport","bring","to","goto"}, Args.Command) then
        local Target = GrabTargets(tostring(Args[1]), speaker)
        if Args.Command == "bring" then
            if type(Target) == "table" then
                for i,v in pairs(Target) do
                    tp(v,speaker)
                end
            else
                tp(Target,speaker)
            end
        elseif Args.Command == "to" or Args.Command == "goto" then
            tp(speaker,Target)
        else
            local Target2 = GrabTargets(tostring(Args[2]), speaker)
            if Target == nil or Target2 == nil then
                return
            end
            if type(Target) == "table" then
                for i,v in pairs(Target) do
                    if Target2 ~= nil then
                        tp(v,Target2)
                    end
                end
            else
                tp(Target,Target2)
            end
        end
    elseif IsInTable({"punish","kick"}, Args.Command) then
        local Target = GrabTargets(tostring(Args[1]), speaker)
        if Target == nil then
            return
        end
        if Target == Player then
            return
        end
        if type(Target) == "table" then
            for i,v in pairs(Target) do
                punish(v)
            end
        else
            punish(Target)
        end
    elseif IsInTable({"god","godmode","infhp","invinsible"}, Args.Command) then
        local Target = GrabTargets(tostring(Args[1]), speaker)
        if Target == nil then
            Target = speaker
        end
        if type(Target) == "table" then
            for i,v in pairs(Target) do
                if not table.find(godded,v.Name) then
                    table.insert(godded,v.Name)
                end
            end
        else
            if not table.find(godded,Target.Name) then
                table.insert(godded,Target.Name)
            end
        end
    elseif IsInTable({"loopgoto","loopto","lg","lt","loopbring","lb","annoy","loopteleport","looptp","loopmove","ltp"}, Args.Command) then
        -- checking for every loop command at once, and gonna separate them inside here
        -- teleporting.PLAYERTOMOVENAME = PLAYERTOGOTONAME
        local Target = nil
        local SecondTarget = nil 
        if Args[1] ~= nil then 
            Target = GrabTargets(Args[1], speaker)
        end
        if Args[2] ~= nil then
            SecondTarget = GrabTargets(Args[2], speaker) 
        end
        if IsInTable({"loopgoto","loopto","lg","lt"}, Args.Command) then -- checking the type of command that was Done
            if type(Target) ~= "table" and check(Target) and check(speaker) then -- if the target exists
                teleporting[speaker.Name] = Target.Name
            end
        elseif IsInTable({"loopbring","lb","annoy"}, Args.Command) then
            if type(Target) == "table" then
                for i,v in pairs(Target) do
                    teleporting[v.Name] = speaker.Name
                end
                return
            end
            if type(Target) ~= "table" and check(Target) and check(speaker) then
                teleporting[Target.Name] = speaker.Name
            end
        elseif IsInTable({"looptp","loopmove","ltp"}, Args.Command) then
            if type(Target) == "table" and type(SecondTarget) ~= "table" then
                for i,v in pairs(Target) do
                    if v ~= speaker and v ~= SecondTarget then
                        teleporting[v.Name] = SecondTarget.Name
                    end
                end
                return
            end
            if type(Target) ~= "table" and type(SecondTarget) ~= "table" and check(Target) and check(SecondTarget) then
                teleporting[Target.Name] = SecondTarget.Name
            end
        end
    elseif IsInTable({"unloopgoto","unloopto","ulg","ult","unloopbring","ulb","unannoy","unloopteleport","unlooptp","unloopmove","ultp"}, Args.Command) then
        if IsInTable({"unloopgoto","unloopto","unlg","ult"}, Args.Command) then
            if Args[1] == nil or Args[1] == "all" or Args[1] == "others" then
                for i,v in pairs(teleporting) do
                    if string.lower(i) == string.lower(speaker.Name) then
                        teleporting[i] = nil
                    end
                end
                return
            end
            for i,v in pairs(teleporting) do
                if string.lower(string.sub(v, 0,#Args[1])) == string.lower(Args[1]) then
                    teleporting[i] = nil
                end
            end
        elseif IsInTable({"unloopbring","unlb","unannoy"}, Args.Command) then
            if Args[1] == nil or Args[1] == "all" or Args[1] == "others" then
                for i,v in pairs(teleporting) do
                    if string.lower(v) == string.lower(speaker.Name) then
                        teleporting[i] = nil
                    end
                end
                return
            end
            for i,v in pairs(teleporting) do
                if string.lower(string.sub(i, 0,#Args[1])) == string.lower(Args[1]) then
                    teleporting[i] = nil 
                end
            end
        elseif IsInTable({"unlooptp","unloopmove","ultp"}, Args.Command) then
            if Args[1] == nil or Args[1] == "all" or Args[1] == "others" then
                for i,v in pairs(teleporting) do
                    teleporting[i] = nil
                end
                return
            end
            
            for i,v in pairs(teleporting) do
                if string.lower(string.sub(i, 0,#Args[1])) == string.lower(Args[1]) or string.lower(string.sub(v, 0,#Args[1])) == string.lower(Args[1]) then
                    teleporting[i] = nil
                end
            end
        end
    elseif IsInTable({"looptps","loopteleporting","teleporting"}, Args.Command) then
        for i,v in pairs(teleporting) do
            print(i,v)
        end
    elseif IsInTable({"ungod","ungodmode","uninfhp"}, Args.Command) then
        local Target = GrabTargets(tostring(Args[1]), speaker)
        if Target == nil then
            Target = speaker
        end
        if type(Target) == "table" then
            for i,v in pairs(Target) do
                if table.find(godded,v.Name) then
                    table.remove(godded,table.find(godded,v.Name))
                end
            end
        else
            if table.find(godded,Target.Name) then
                table.remove(godded,table.find(godded,Target.Name))
            end
        end
    elseif IsInTable({"aura","killaura","kaura"}, Args.Command) then
        local Target = GrabTargets(tostring(Args[1]), speaker)
        if Target == nil and Args[1] ~= nil then
            return
        end
        if Args[1] == nil then
            Target = speaker
        end
        if type(Target) == "table" then
            for i,v in pairs(Target) do
                if not table.find(aura,v.Name) then
                    table.insert(aura,v.Name)
                    repeat wait()
                        for i2,v2 in pairs(Players:GetPlayers()) do
                            if check(v2) and check(v) and v2 ~= Player and v2 ~= v then
                                if (v2.Character.HumanoidRootPart.Position - v.Character.HumanoidRootPart.Position).Magnitude < 20 then 
                                    kill(v2)
                                end
                            end
                        end
                    until not table.find(aura,v.Name)
                end
            end
        else
            if not table.find(aura,Target.Name) then
                table.insert(aura,Target.Name)
                repeat wait()
                    for i3,v3 in pairs(Players:GetPlayers()) do
                        if check(v3) and check(Target) and v3 ~= Player and v3 ~= Target then
                            if (v3.Character.HumanoidRootPart.Position - Target.Character.HumanoidRootPart.Position).Magnitude < 20 then 
                                kill(v3)
                            end
                        end
                    end
                until not table.find(aura,Target.Name)
            end
        end
    elseif IsInTable({"unaura","unkillaura","unkaura"}, Args.Command) then
        local Target = GrabTargets(tostring(Args[1]), speaker)
        if Target == nil then
            return
        end
        if type(Target) == "table" then
            for i,v in pairs(Target) do
                if table.find(aura,v.Name) then
                    table.remove(aura,table.find(aura,v.Name))
                end
            end
        else
            if table.find(aura,Target.Name) then
                table.remove(aura,table.find(aura,Target.Name))
            end
        end
    elseif IsInTable({"swap"}, Args.Command) then
        local Target = GrabTargets(tostring(Args[1]), speaker)
        local Target2 = GrabTargets(tostring(Args[2]), speaker)
        if Target == nil then
            return
        end
        if Target2 == nil then
			Target2 = speaker
		end
		if type(Target) == "table" then
			for i,v in pairs(Target) do
				if Target2 ~= nil then
					swap(v,Target2)
				end
			end
		else
			swap(Target,Target2)
		end
    elseif IsInTable({"arena","fightnight","battleroyale","fightforadmin"}, Args.Command) then
        if check(speaker) and check(Player) then
            MainRemote:FireServer("Module","SetCFrame",speaker.Character.HumanoidRootPart,CFrame.new(1875.4908447266, 2400.255859375, 750.27435302734))
            MainRemote:FireServer("Module","SetCFrame",Player.Character.HumanoidRootPart,CFrame.new(1875.4908447266, 2400.255859375, 750.27435302734))
        end
        wait(0.5)
        rocks()
        for i,v in pairs(game.Players:GetPlayers()) do
            if check(v) and v ~= speaker then
                if v.Character:FindFirstChildOfClass("Humanoid") and v.Character:FindFirstChild("HumanoidRootPart") then
                    MainRemote:FireServer("Module","SetCFrame",v.Character.HumanoidRootPart,CFrame.new(1872, 2150, 548))
                end
            end
        end
    elseif IsInTable({"1v1","1vs1","battle","fight"}, Args.Command) then
        local Target = GrabTargets(tostring(Args[1]), speaker)
        local Target2 = GrabTargets(tostring(Args[2]), speaker)
        if Target2 == nil then
			Target2 = speaker
		end
		if check(Target) and check(Target2) then
			MainRemote:FireServer("Module","SetCFrame",Target.Character.HumanoidRootPart,CFrame.new(1872, 2150, 548))
			MainRemote:FireServer("Module","SetCFrame",Target2.Character.HumanoidRootPart,CFrame.new(1872, 2150, 548))
		end
		wait(0.5)
		rocks()
    elseif IsInTable({"freeze","stop","stun","permstun","infstun"}, Args.Command) then
        local Target = GrabTargets(tostring(Args[1]), speaker)
        if Target == nil then
            return
        end
        if Target == Player then
            return
        end
        if type(Target) == "table" then
            for i,v in pairs(Target) do
                freeze(v)
            end
        else
            freeze(Target)
        end
    elseif IsInTable({"unfreeze","unstop","unstun","unpermstun","uninfstun","uf"}, Args.Command) then
        local Target = GrabTargets(tostring(Args[1]), speaker)
        if Target == nil then
            return
        end
        if type(Target) == "table" then
            for i,v in pairs(Target) do
                unfreeze(v)
            end
        else
            unfreeze(Target)
        end
    elseif IsInTable({"changer","changergui"}, Args.Command) and speaker == Player and Args[1] == nil then
        loadstring(game:HttpGet(('https://raw.githubusercontent.com/XTheMasterX/Scripts/Main/SlayersChanger'),true))()
    elseif IsInTable({"nocooldown","ncd","nocool"}, Args.Command) and speaker == Player and Args[1] == nil then
        for i,v in pairs(getrenv()._G.Moves) do
            if type(v) == "table" then
                for i2,v2 in pairs(v.Moves) do
                    if type(v2) == "table" and v2.Name ~= nil and v2.Cooldown ~= nil then
                        if oldcool[v2.Name] == nil then
                            oldcool[v2.Name] = v2.Cooldown
                        end
                        v2.Cooldown = 0
                    end
                end
            end
        end
        game.StarterGui:SetCore("SendNotification", {
            Title = "No CoolDown!";
            Text = "No CoolDown is ON!";
            Icon = "rbxassetid://2541869220";
            Duration = 3;
        })
    elseif IsInTable({"unnocooldown","unncd","unnocool","cooldown","cd"}, Args.Command) and speaker == Player and Args[1] == nil then
        for i,v in pairs(getrenv()._G.Moves) do
            if type(v) == "table" then
                for i2,v2 in pairs(v.Moves) do
                    if type(v2) == "table" and v2.Name ~= nil and v2.Cooldown ~= nil then
                        if oldcool[v2.Name] ~= nil then
                            v2.Cooldown = oldcool[v2.Name] 
                        end
                    end
                end
            end
        end
        game.StarterGui:SetCore("SendNotification", {
            Title = "No CoolDown!";
            Text = "No CoolDown is OFF!";
            Icon = "rbxassetid://2541869220";
            Duration = 3;
        }) 
    elseif IsInTable({"rj","rejoin"}, Args.Command) and speaker == Player and Args[1] == nil then
        game:GetService('TeleportService'):TeleportToPlaceInstance(game.PlaceId, game.JobId)
    elseif IsInTable({"ruiplace","rui","ruirobe"}, Args.Command) and speaker == Player and Args[1] == nil then
        if Args.Command == "rui" then
            game:GetService("TeleportService"):Teleport(7146829768)
            game.StarterGui:SetCore("SendNotification", {
                Title = "Teleport!";
                Text = "Teleporting to Rui Boss!";
                Icon = "rbxassetid://2541869220";
                Duration = 3;
            })
        elseif Args.Command == "ruiplace" then
            Player.Character.HumanoidRootPart.CFrame = CFrame.new(1804, 1930.0289306641, 6322)
        elseif Args.Command == "ruirobe" then
            table.insert(getrenv()._G.Data.Inventory.Items,{
                ["Id"] = 1001,
                ["Type"] = "Clothing",
                ["Name"] = "Rui's Robe",
                ["Rarity"] = "Unique"
            })
            save()
            game.StarterGui:SetCore("SendNotification", {
                Title = "Items!";
                Text = "Added Rui's Robe To Your Inventory!";
                Icon = "rbxassetid://2541869220";
                Duration = 3;
            }) 
        end
    elseif IsInTable({"fs","finalselection","finalplace","fsi","finalselectionitems","finalitems"}, Args.Command) and speaker == Player and Args[1] == nil then
        if IsInTable({"fs","finalselection","finalplace"}, Args.Command) then
            Player.Character.HumanoidRootPart.CFrame = CFrame.new(3510.1813964844, 1558.6177978516, 1335.7103271484)
        else
            table.insert(getrenv()._G.Data.Inventory.Items,{
                ["Rarity"] = "Mythic", 
            	["Id"] = 10, 
            	["Name"] = "Nichirin Blade", 
            	["Type"] = "Tool", 
            	["Health"] =  {
            	[1] = 0, 
            	[2] = 5},
            	["Stats"] =  {
            	["Strength"] =  {
            	[1] = 5, 
            	[2] = 100}, 
            	["Defense"] =  {
            	[1] = 20, 
            	[2] = 100}}
            	})
            table.insert(getrenv()._G.Data.Inventory.Items,{
            	["Id"] = 4659180138, 
            	["Type"] = "Shirt", 
            	["Name"] = "Demon Slayer Corps Top", 
            	["Rarity"] = "Common"
                })
            table.insert(getrenv()._G.Data.Inventory.Items,{
            	["Id"] = 4659183633, 
            	["Type"] = "Pants", 
            	["Name"] = "Demon Slayer Corps Bottom", 
            	["Rarity"] = "Common"
                })
            getrenv()._G.Data.CompletedFinalSelection = true
            save()
            game.StarterGui:SetCore("SendNotification", {
                Title = "Items!";
                Text = "Added Final Selection Items To Your Inventory!";
                Icon = "rbxassetid://2541869220";
                Duration = 3;
            })
        end
    elseif IsInTable({"unlock","unlockmoves","unlockskills"}, Args.Command) and speaker == Player and Args[1] == nil then
        for i,v in pairs(getrenv()._G.Moves) do
            if type(v) == "table" then
                for i2,v2 in pairs(v.Moves) do
                    if v2.Name then
                        if not table.find(movey,v2.Name) then
                            table.insert(movey,v2.Name)
                        end
                    end
                end
            end
        end
        local movesy = getrenv()._G.Data.UnlockedMoves
        for i,v in pairs(movey) do
            if not table.find(movesy,v) then
                table.insert(movesy,v)
            end
        end
        save()
        game.StarterGui:SetCore("SendNotification", {
            Title = "Moves!";
            Text = "Unlocked All Moves!";
            Icon = "rbxassetid://2541869220";
            Duration = 3;
        })
    end 
end

local mouse = Player:GetMouse()

local function Closest()
    local LowestDistance = math.huge
    local ClosestCharacter = nil
    local Characters = workspace.Mobs:GetChildren()
    for i,v in pairs(game:GetService("Players"):GetPlayers()) do
        if v.Character ~= nil and v ~= game.Players.LocalPlayer then
            table.insert(Characters, v.Character)
        end
    end
    for i,v in pairs(Characters) do
        if v ~= nil then
            if v:FindFirstChild("HumanoidRootPart") then
                local InitialDis = (v.HumanoidRootPart.Position - game.Workspace.CurrentCamera.CFrame.p).magnitude
                local Ray = Ray.new(game.Workspace.CurrentCamera.CFrame.p, (mouse.Hit.p - game.Workspace.CurrentCamera.CFrame.p).unit * InitialDis)
                local Part,Position = game.Workspace:FindPartOnRay(Ray, game.Workspace)
                local FinalDifference = math.floor((Position - v.HumanoidRootPart.Position).magnitude)
                if FinalDifference < LowestDistance then
                    ClosestCharacter = v
                    LowestDistance = FinalDifference
                end
            end
        end
    end
    local ClosestPlayer = game.Players:GetPlayerFromCharacter(ClosestCharacter)
    if ClosestPlayer ~= nil then
        return ClosestPlayer
    else
        return ClosestCharacter
    end
end

mouse.KeyDown:Connect(function(key)
    if key:lower() == "r" then
        local v = Closest()
        if v then
            kill(v)
        end
	elseif key:lower() == "g" then
        togglef3x()
    elseif key:lower() == "t" then
        game:GetService("ReplicatedStorage").Remotes.DamageHandler:FireServer(Player.Character.Humanoid)
        MainRemote:FireServer("Module", "TakeDamage", Player.Character.Humanoid, Player.Character.Humanoid.Health - Player.Character.Humanoid.MaxHealth)
    end
end)

Player.Chatted:Connect(function(msg)
    CommandHandler(msg, Player)
end)

Players.PlayerAdded:Connect(function(plr)
    if IsInTable(Whitelist, plr.Name) then
        SetUpUsers[plr.Name] = true
        plr.Chatted:Connect(function(msg)
            CommandHandler(msg, plr)
        end)
        wait(1)
        if Player.PlayerGui:FindFirstChild("Leaderboard") then
            for i,v in pairs(Player.PlayerGui.Leaderboard:GetDescendants()) do
                if v.Name == "name" and v:FindFirstAncestor(plr.Name) then
                    v.TextColor3 = Color3.new(1, 170/255, 0)
                end
            end
        end
    end
end)

Players.PlayerRemoving:Connect(function(plr)
    local plrname = plr.Name
    if IsInTable(SetUpUsers, plrname) then
        SetUpUsers[plrname] = nil
    end
    if teleporting[plrname] ~= nil then
        teleporting[plrname] = nil
    end
end)

game:GetService('RunService').Stepped:connect(function()
	if F3xEnabled then
        for i,v in pairs(Player.Backpack:GetChildren()) do
            if v.Name == "F3X" then
                v.Parent = Player.Character
            end
        end
    end
    if #killing ~= 0 then
        for i = 1,#killing do
            if game.Players:FindFirstChild(killing[i]) then
                kill(game.Players[killing[i]])
            end
        end
    end
    if #voiding ~= 0 then
        for i = 1,#voiding do
            if game.Players:FindFirstChild(voiding[i]) then
                void(game.Players[voiding[i]])
            end
        end
    end
    if #godded ~= 0 then
        for i = 1,#godded do
            if game.Players:FindFirstChild(godded[i]) then
                god(game.Players[godded[i]])
            end
        end
    end
    for i,v in pairs(teleporting) do
        if game.Players:FindFirstChild(i) and game.Players:FindFirstChild(v) then
            tp(game.Players[i],game.Players[v])
        end
    end
end)

print([[
    
!! shortened names and silent commands are both possible! (/e kill PLAYER) !!
---------------------- Hotkeys -------------------------
R Key -- Kills closest player or mob to the cursor
T Key -- Heals you
G Key -- Toggles f3x
---------------------- Commands! -----------------------
:whitelist player -- Gives player commands
:whitelist others -- Gives everyone else commands
:whitelist random -- Gives a random player commands
ALIASES: | :wl | :admin | :givecommands | :rank |

:unwhitelist player -- Takes player's commands
:unwhitelist others -- Takes everyone else's commands
:unwhitelist random -- Takes a random player's commands
ALIASES: | :unwl | :unadmin | :ungivecommands | :takecommands | :unrank |

:wls -- Prints a list of whitelisted players
ALIASES: | :printwhitelists | :whitelists | :printwl | :wllist | :admins |

:kill player -- Kills player
:kill others -- Kills everyone else
:kill all -- Kills everyone
:kill random -- Kills a random player
:kill mobs -- Kills all mobs
ALIASES: | :murder | :erase | :dump | :eliminate | :end | :destroy |

:heal player -- Fully heals player
:heal others -- Fully heals everyone else
:heal all -- Fully heals everyone
:heal random -- Fully heals a random player
ALIASES: | :hp | :help | :givehp | :full |

:freeze player -- Stops player
:freeze others -- Stops everyone else
:freeze all -- Stops everyone
:freeze random -- Stops a random player
ALIASES: | :stop | :stun | :infstun | :permstun |

:bring player -- Teleports player to you
:bring others -- Teleports everyone else to you
:bring random -- Teleports a random player to you

:goto player -- Teleports you to player
:goto random -- Teleports you to a random player
ALIASE: | :to |

:tp player1 player2 -- Teleports player1 to player2
:tp others player -- Teleports everyone else to player
ALIASE: | :teleport |

:swap player1 player2 -- Swaps player1's place with player2

:god player -- Gives player inf hp
:god others -- Gives everyone else inf hp
:god all -- Gives everyone inf hp
:god random -- Gives a random player inf hp
ALIASES: | :godmode | :infhp | :invincible |

:fling player -- Flings player
:fling others -- Flings others
:fling random -- Flings random
ALIASES: | :void | :space | :sky |

:punish player -- Sends player to the sky and they cant reset (Cant be stopped)
:punish others -- Sends everyone else to the sky and they cant reset (Cant be stopped)
:punish random -- Sends a random player to the sky and they cant reset (Cant be stopped)
ALIASE: | :kick |

:lk player -- Loopkills player
:lk others -- Loopkills everyone else
:lk random -- Loopkills a random player
ALIASES: | :loopkill | :loop | :infkill |

:unlk player -- unloopkills player
:unlk others -- unloopkills everyone else
ALIASES: | :unloopkill | :unloop | :uninfkill |

:lb player -- Loopbrings player
:lb others -- Loopbrings everyone else
:lb random -- Loopbrings a random player
ALIASES: | :loopbring | :annoy |

:unlb player -- unloopbrings player
:unlb others -- unloopbrings everyone else
ALIASES: | :unloopbring | :unannoy |

:lt player -- Loopteleports you to player
:lt random -- Loopteleports you to a random player
ALIASES: | :loopto | :loopgoto | :lg |

:unlt player -- unloopteleports you to player
:unlt random -- unloopteleports you to a random player
ALIASES: | :unloopto | :unloopgoto | :unlg |

:ltp player1 player2 -- Loopteleports player1 to player2
:ltp others player -- Loopteleports everyone else to player
ALIASES: | :looptp | :loopmove |

:lf player -- Loopflings player
:lf others -- Loopflings everyone else
:lf random -- Loopflings a random player
ALIASES: | :loopfling | :loopvoid | :loopspace | :lv | :ls |

:unlf player -- unloopflings player
:unlf others -- unloopflings everyone else
ALIASES: | :unloopfling | :unloopvoid | :unloopspace | :unlv | :unls |

:1v1 player -- Teleports you and player to the arena
:1v1 player1 player2 -- Teleports player1 and player2 to the arena
ALIASES: | :1vs1 | :fight | :battle |

:aura player -- Gives player kill aura
:aura others -- Gives everyone else kill aura
:aura random -- Gives a random player kill aura
ALIASES: | :killaura | :kaura |

:unaura player -- Takes player's kill aura
:unaura others -- Takes everyone else's kill aura
ALIASES: | :unkillaura | :unkaura |

:arena -- Teleports everyone to a battle royale arena
ALIASES: | :fightnight | :battleroyale | :fightforadmin |

:nocooldown -- Removes cooldowns from your moves
ALIASES : | :ncd | :nocool |

:cooldown -- Adds the cooldowns back
ALIASES : | :unncd | :unnocool | :cd | :cooldown

:unlock -- Unlocks all moves
ALIASES : | :unlockmoves | :unlockskills |

:rui -- Teleports you to rui's server

:ruiplace -- Teleports you to rui's circle

:ruirobe -- Gives you rui's robe

:finalselection -- Teleports you to the final selection npc
ALIASES : | :fs | :finalplace |

:finalselectionitems -- Gives you Final selection items
ALIASES : | :fsi | :finalitems |

:rejoin -- Rejoins the same server your in
ALIASE : | :rj |

:changer -- Loads changer gui

Enjoy!
-------------------------------------------------------
----------------Made By Aidez & Septex-----------------
-------------------------------------------------------
]])

repeat wait() until Player.Character ~= nil
Player.Character.ChildAdded:Connect(function(child)
    if child:IsA("Tool") and F3xEnabled then
        wait()
        child.Parent = Player.Backpack
    end
end)
Player.CharacterAdded:Connect(function(char)
    char.ChildAdded:Connect(function(child)
        if child:IsA("Tool") and F3xEnabled then
            wait()
            child.Parent = Player.Backpack
        end
    end)
end)

game:GetService("StarterGui"):SetCore("SendNotification", {
    Title = "Loaded!",
    Text = "Slayer Commands Loaded! Made By Aidez & Septex",
    Icon = "rbxthumb://type=Asset&id=2022095751&w=420&h=420",
    Duration = 5,
})
wait(3)
game:GetService("StarterGui"):SetCore("SendNotification", {
    Title = "Commands!",
    Text = "Press Shift + F9 To View The Commands!",
    Icon = "rbxthumb://type=Asset&id=5396227392&w=420&h=420",
    Duration = 5,
}) 
