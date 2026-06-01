//@version=4
strategy("TSA TDI Double Divergence without levels Strategy", overlay = true, max_bars_back = 5000, default_qty_type = strategy.percent_of_equity, default_qty_value = 100)

// Strategy Parameters
prd = input(defval = 5, title = "Pivot Period", minval = 1, maxval = 50)
source = input(defval = "Close", title = "Source for Pivot Points", options = ["Close", "High/Low"])
searchdiv = "Regular" // Only use Regular divergences
showindis = input(defval = "Full", title = "Show Indicator Names", options = ["Full", "First Letter", "Don't Show"])
maxpp = input(defval = 10, title = "Maximum Pivot Points to Check", minval = 1, maxval = 20)
maxbars = input(defval = 100, title = "Maximum Bars to Check", minval = 30, maxval = 200)
shownum = input(defval = true, title = "Show Divergence Number")
showlast = input(defval = false, title = "Show Only Last Divergence")
dontconfirm = input(defval = false, title = "Don't Wait for Confirmation")
showlines = input(defval = true, title = "Show Divergence Lines")
showpivot = input(defval = false, title = "Show Pivot Points")
showmas = input(defval = false, title = "Show MAs 50 & 200", inline = "ma12")
cma1col = input(defval = color.lime, title = "", inline = "ma12")
cma2col = input(defval = color.red, title = "", inline = "ma12")

// TSA TDI Indicator inputs
show_tdi = input(defval = true, title = "Show TSA TDI on Chart")
tdi_rsi_length = input(defval = 8, title = "TSA TDI RSI Length", minval = 1)
tdi_ma_length = input(defval = 34, title = "TSA TDI MA Length", minval = 1)
tdi_fast_length = input(defval = 2, title = "TSA TDI Fast MA Length", minval = 1)

// Strategy specific parameters
use_stop_loss = input(defval = true, title = "Use Stop Loss")
stop_loss_pips = input(defval = 30, title = "Stop Loss (pips)", minval = 1, step = 1)
use_take_profit = input(defval = true, title = "Use Take Profit")
take_profit_pips = input(defval = 30, title = "Take Profit (pips)", minval = 1, step = 1)
// We only trade on regular divergences
trade_on_reg_div = true

// Colors for divergences
pos_reg_div_col = input(defval = color.yellow, title = "Positive Regular Divergence")
neg_reg_div_col = input(defval = color.navy, title = "Negative Regular Divergence")
pos_hid_div_col = input(defval = color.lime, title = "Positive Hidden Divergence")
neg_hid_div_col = input(defval = color.red, title = "Negative Hidden Divergence")
pos_div_text_col = input(defval = color.black, title = "Positive Divergence Text Color")
neg_div_text_col = input(defval = color.white, title = "Negative Divergence Text Color")

// Line styles
reg_div_l_style_ = input(defval = "Solid", title = "Regular Divergence Line Style", options = ["Solid", "Dashed", "Dotted"])
hid_div_l_style_ = input(defval = "Dashed", title = "Hidden Divergence Line Style", options = ["Solid", "Dashed", "Dotted"])
reg_div_l_width = input(defval = 2, title = "Regular Divergence Line Width", minval = 1, maxval = 5)
hid_div_l_width = input(defval = 1, title = "Hidden Divergence Line Width", minval = 1, maxval = 5)

plot(showmas ? sma(close, 50) : na, color = showmas ? cma1col : na)
plot(showmas ? sma(close, 200) : na, color = showmas ? cma2col: na)

// Set line styles
var reg_div_l_style = reg_div_l_style_ == "Solid" ? line.style_solid : 
                       reg_div_l_style_ == "Dashed" ? line.style_dashed :
                       line.style_dotted
var hid_div_l_style = hid_div_l_style_ == "Solid" ? line.style_solid : 
                       hid_div_l_style_ == "Dashed" ? line.style_dashed :
                       line.style_dotted

// TSA TDI Indicator calculation
tdi_rsi = rsi(close, tdi_rsi_length)
tdi_ma = sma(tdi_rsi, tdi_ma_length)
tdi_offs = 1.6185 * stdev(tdi_rsi, tdi_ma_length)
tdi_up = tdi_ma + tdi_offs
tdi_dn = tdi_ma - tdi_offs
tdi_mid = (tdi_up + tdi_dn) / 2
tdi_fastMA = sma(tdi_rsi, tdi_fast_length)

// Display TSA TDI if enabled
var show_ob = show_tdi ? 75 : na
var show_os = show_tdi ? 25 : na
var show_h80 = show_tdi ? 80 : na
var show_h50 = show_tdi ? 50 : na
var show_h20 = show_tdi ? 20 : na

hline(show_ob, "OverBought", color=color.gray, linestyle=hline.style_dashed)
hline(show_os, "OverSold", color=color.gray, linestyle=hline.style_dashed)
hline(show_h80, "Horizontal Line", color=color.gray, linestyle=hline.style_dashed)
hline(show_h50, "Horizontal Line", color=color.gray, linestyle=hline.style_dashed)
hline(show_h20, "Horizontal Line", color=color.gray, linestyle=hline.style_dashed)

// Keep indicator name and colors in arrays
var indicators_name = array.new_string(1)
var div_colors = array.new_color(4)
if barstate.isfirst
    // Name
    array.set(indicators_name, 0, showindis == "Full" ? "TDI" : "T")
    // Colors
    array.set(div_colors, 0, pos_reg_div_col)
    array.set(div_colors, 1, neg_reg_div_col)
    array.set(div_colors, 2, pos_hid_div_col)
    array.set(div_colors, 3, neg_hid_div_col)

// Check if we get new Pivot High Or Pivot Low
float ph = pivothigh((source == "Close" ? close : high), prd, prd)
float pl = pivotlow((source == "Close" ? close : low), prd, prd)
plotshape(ph and showpivot, text = "H", style = shape.labeldown, color = color.new(color.white, 100), textcolor = color.red, location = location.abovebar, offset = -prd)
plotshape(pl and showpivot, text = "L", style = shape.labelup, color = color.new(color.white, 100), textcolor = color.lime, location = location.belowbar, offset = -prd)

// Keep values and positions of Pivot Highs/Lows in the arrays
var int maxarraysize = 20
var ph_positions = array.new_int(maxarraysize, 0)
var pl_positions = array.new_int(maxarraysize, 0)
var ph_vals = array.new_float(maxarraysize, 0.)
var pl_vals = array.new_float(maxarraysize, 0.)

// Add PHs to the array
if ph
    array.unshift(ph_positions, bar_index)
    array.unshift(ph_vals, ph)
    if array.size(ph_positions) > maxarraysize
        array.pop(ph_positions)
        array.pop(ph_vals)

// Add PLs to the array
if pl
    array.unshift(pl_positions, bar_index)
    array.unshift(pl_vals, pl)
    if array.size(pl_positions) > maxarraysize
        array.pop(pl_positions)
        array.pop(pl_vals)

// Functions to check Regular Divergences and Hidden Divergences

// Function to check positive regular or negative hidden divergence
// cond == 1 => positive_regular, cond == 2=> negative_hidden
positive_regular_positive_hidden_divergence(src, cond)=>
    divlen = 0
    prsc = source == "Close" ? close : low
    // If indicators higher than last value and close price is higher than last close 
    if dontconfirm or src > src[1] or close > close[1]
        startpoint = dontconfirm ? 0 : 1 // Don't check last candle
        // We search last maxpp PPs
        for x = 0 to maxpp - 1
            // Add protection against accessing non-existent array elements
            if x >= array.size(pl_positions)
                break
            
            len = bar_index - array.get(pl_positions, x) + prd
            // If we reach non valued array element or arrived maxbars previous bars then we don't search more
            if array.get(pl_positions, x) == 0 or len > maxbars or len <= 0
                break
                
            // Make sure we're not accessing out-of-bounds array indices
            if len <= bar_index and len > 0 and startpoint < bar_index
                if len > 5 and 
                   ((cond == 1 and src[startpoint] > src[len] and prsc[startpoint] < nz(array.get(pl_vals, x))) or
                   (cond == 2 and src[startpoint] < src[len] and prsc[startpoint] > nz(array.get(pl_vals, x))))
                    slope1 = (src[startpoint] - src[len]) / (len - startpoint)
                    virtual_line1 = src[startpoint] - slope1
                    slope2 = (close[startpoint] - close[len]) / (len - startpoint)
                    virtual_line2 = close[startpoint] - slope2
                    arrived = true
                    for y = 1 + startpoint to len - 1
                        if y <= bar_index and y >= 0
                            if src[y] < virtual_line1 or nz(close[y]) < virtual_line2
                                arrived := false
                                break
                        virtual_line1 := virtual_line1 - slope1
                        virtual_line2 := virtual_line2 - slope2
                    
                    if arrived
                        divlen := len
                        break
    divlen

// Function to check negative regular or positive hidden divergence
// cond == 1 => negative_regular, cond == 2=> positive_hidden
negative_regular_negative_hidden_divergence(src, cond)=>
    divlen = 0
    prsc = source == "Close" ? close : high
    // If indicators higher than last value and close price is higher than last close 
    if dontconfirm or src < src[1] or close < close[1]
        startpoint = dontconfirm ? 0 : 1 // Don't check last candle
        // We search last maxpp PPs
        for x = 0 to maxpp - 1
            // Add protection against accessing non-existent array elements
            if x >= array.size(ph_positions)
                break
                
            len = bar_index - array.get(ph_positions, x) + prd
            // If we reach non valued array element or arrived maxbars previous bars then we don't search more
            if array.get(ph_positions, x) == 0 or len > maxbars or len <= 0
                break
                
            // Make sure we're not accessing out-of-bounds array indices
            if len <= bar_index and len > 0 and startpoint < bar_index
                if len > 5 and 
                   ((cond == 1 and src[startpoint] < src[len] and prsc[startpoint] > nz(array.get(ph_vals, x))) or 
                   (cond == 2 and src[startpoint] > src[len] and prsc[startpoint] < nz(array.get(ph_vals, x))))
                    slope1 = (src[startpoint] - src[len]) / (len - startpoint)
                    virtual_line1 = src[startpoint] - slope1
                    slope2 = (close[startpoint] - nz(close[len])) / (len - startpoint)
                    virtual_line2 = close[startpoint] - slope2
                    arrived = true
                    for y = 1 + startpoint to len - 1
                        if y <= bar_index and y >= 0
                            if src[y] > virtual_line1 or nz(close[y]) > virtual_line2
                                arrived := false
                                break
                        virtual_line1 := virtual_line1 - slope1
                        virtual_line2 := virtual_line2 - slope2
                    
                    if arrived
                        divlen := len
                        break
    divlen

// Calculate 4 types of divergence and return divergences in an array
calculate_divs(indicator)=>
    divs = array.new_int(4, 0)
    array.set(divs, 0, (searchdiv == "Regular" or searchdiv == "Regular/Hidden") ? positive_regular_positive_hidden_divergence(indicator, 1) : 0)
    array.set(divs, 1, (searchdiv == "Regular" or searchdiv == "Regular/Hidden") ? negative_regular_negative_hidden_divergence(indicator, 1) : 0)
    array.set(divs, 2, (searchdiv == "Hidden" or searchdiv == "Regular/Hidden") ? positive_regular_positive_hidden_divergence(indicator, 2) : 0)
    array.set(divs, 3, (searchdiv == "Hidden" or searchdiv == "Regular/Hidden") ? negative_regular_negative_hidden_divergence(indicator, 2) : 0)
    divs

// Array to keep all divergences (4 types)
var all_divergences = array.new_int(4)

// Calculate TSA TDI divergences and store in array
tdi_divs = calculate_divs(tdi_fastMA)
for i = 0 to 3
    array.set(all_divergences, i, array.get(tdi_divs, i))

// Keep lines in arrays
var pos_div_lines = array.new_line(0)
var neg_div_lines = array.new_line(0)
var pos_div_labels = array.new_label(0)
var neg_div_labels = array.new_label(0) 

// Remove old lines and labels if showlast option is enabled
delete_old_pos_div_lines()=>
    if array.size(pos_div_lines) > 0    
        for j = 0 to array.size(pos_div_lines) - 1 
            line.delete(array.get(pos_div_lines, j))
        array.clear(pos_div_lines)

delete_old_neg_div_lines()=>
    if array.size(neg_div_lines) > 0    
        for j = 0 to array.size(neg_div_lines) - 1 
            line.delete(array.get(neg_div_lines, j))
        array.clear(neg_div_lines)

delete_old_pos_div_labels()=>
    if array.size(pos_div_labels) > 0 
        for j = 0 to array.size(pos_div_labels) - 1 
            label.delete(array.get(pos_div_labels, j))
        array.clear(pos_div_labels)

delete_old_neg_div_labels()=>
    if array.size(neg_div_labels) > 0    
        for j = 0 to array.size(neg_div_labels) - 1 
            label.delete(array.get(neg_div_labels, j))
        array.clear(neg_div_labels)

// Delete last created lines and labels until we meet new PH/PV 
delete_last_pos_div_lines_label(n)=>
    if n > 0 and array.size(pos_div_lines) >= n    
        asz = array.size(pos_div_lines)
        for j = 1 to n
            line.delete(array.get(pos_div_lines, asz - j))
            array.pop(pos_div_lines)
        if array.size(pos_div_labels) > 0  
            label.delete(array.get(pos_div_labels, array.size(pos_div_labels) - 1))
            array.pop(pos_div_labels)

delete_last_neg_div_lines_label(n)=>
    if n > 0 and array.size(neg_div_lines) >= n    
        asz = array.size(neg_div_lines)
        for j = 1 to n
            line.delete(array.get(neg_div_lines, asz - j))
            array.pop(neg_div_lines)
        if array.size(neg_div_labels) > 0  
            label.delete(array.get(neg_div_labels, array.size(neg_div_labels) - 1))
            array.pop(neg_div_labels)
            
// Variables for Divergence Detection
pos_reg_div_detected = false
neg_reg_div_detected = false
pos_hid_div_detected = false
neg_hid_div_detected = false

// To remove lines/labels until we meet new PH/PL
var last_pos_div_lines = 0
var last_neg_div_lines = 0
var remove_last_pos_divs = false 
var remove_last_neg_divs = false
if pl
    remove_last_pos_divs := false
    last_pos_div_lines := 0
if ph
    remove_last_neg_divs := false
    last_neg_div_lines := 0

// Draw divergences lines and labels
divergence_text_top = ""
divergence_text_bottom = ""
distances = array.new_int(0)
dnumdiv_top = 0
dnumdiv_bottom = 0
top_label_col = color.white
bottom_label_col = color.white
old_pos_divs_can_be_removed = true
old_neg_divs_can_be_removed = true
startpoint = dontconfirm ? 0 : 1 // Used for don't confirm option

// Variables to store TDI values at divergence points for the filter conditions
var float tdi_value_at_div_start = na
var float tdi_value_at_div_end = na
var int div_start_bar_index = 0

// Process all 4 divergence types
for y = 0 to 3
    if array.get(all_divergences, y) > 0 // Any divergence?
        div_len = array.get(all_divergences, y)
        
        // Safety check to make sure we don't access data beyond available bars
        if div_len <= bar_index
            div_start_bar_index := bar_index - div_len
            
            // Store the TDI value at the start of the divergence (first leg)
            if y == 0 and div_len < bar_index  // For positive regular divergence
                tdi_value_at_div_start := tdi_fastMA[div_len]
            if y == 1 and div_len < bar_index  // For negative regular divergence
                tdi_value_at_div_start := tdi_fastMA[div_len]
                
            if (y % 2) == 1 
                dnumdiv_top := dnumdiv_top + 1
                top_label_col := array.get(div_colors, y)
            if (y % 2) == 0
                dnumdiv_bottom := dnumdiv_bottom + 1
                bottom_label_col := array.get(div_colors, y)
            if not array.includes(distances, array.get(all_divergences, y))  // Line not exist?
                array.push(distances, array.get(all_divergences, y))
                
                // Safety checks for line drawing
                line_ok = div_len <= bar_index and startpoint <= bar_index
                
                new_line = showlines and line_ok ? line.new(
                          x1 = bar_index - div_len, 
                          y1 = (source == "Close" ? close[div_len] : 
                                           (y % 2) == 0 ? low[div_len] : 
                                                          high[div_len]),
                          x2 = bar_index - startpoint,
                          y2 = (source == "Close" ? close[startpoint] : 
                                           (y % 2) == 0 ? low[startpoint] : 
                                                          high[startpoint]),
                          color = array.get(div_colors, y),
                          style = y < 2 ? reg_div_l_style : hid_div_l_style,
                          width = y < 2 ? reg_div_l_width : hid_div_l_width
                          )
                          : na
                if (y % 2) == 0
                    if old_pos_divs_can_be_removed
                        old_pos_divs_can_be_removed := false
                        if not showlast and remove_last_pos_divs
                            delete_last_pos_div_lines_label(last_pos_div_lines)
                            last_pos_div_lines := 0
                        if showlast
                            delete_old_pos_div_lines()
                    if not na(new_line)  // Only add valid line objects
                        array.push(pos_div_lines, new_line)
                        last_pos_div_lines := last_pos_div_lines + 1
                        remove_last_pos_divs := true
                    
                if (y % 2) == 1
                    if old_neg_divs_can_be_removed
                        old_neg_divs_can_be_removed := false
                        if not showlast and remove_last_neg_divs
                            delete_last_neg_div_lines_label(last_neg_div_lines)
                            last_neg_div_lines := 0
                        if showlast
                            delete_old_neg_div_lines()
                    if not na(new_line)  // Only add valid line objects
                        array.push(neg_div_lines, new_line)
                        last_neg_div_lines := last_neg_div_lines + 1
                        remove_last_neg_divs := true
                    
            // Set variables for trade signals
            if y == 0
                pos_reg_div_detected := true
            if y == 1
                neg_reg_div_detected := true
            if y == 2
                pos_hid_div_detected := true
            if y == 3
                neg_hid_div_detected := true
                
            // Get text for labels  
            if showindis != "Don't Show"
                divergence_text_top := divergence_text_top + ((y % 2) == 1 ? array.get(indicators_name, 0) + "\n" : "")
                divergence_text_bottom := divergence_text_bottom + ((y % 2) == 0 ? array.get(indicators_name, 0) + "\n" : "")

// Draw labels
if showindis != "Don't Show" or shownum
    if shownum and dnumdiv_top > 0
        divergence_text_top := divergence_text_top + tostring(dnumdiv_top)
    if shownum and dnumdiv_bottom > 0
        divergence_text_bottom := divergence_text_bottom + tostring(dnumdiv_bottom)
    if divergence_text_top != ""
        if showlast
            delete_old_neg_div_labels()
        array.push(neg_div_labels, 
                      label.new( x = bar_index, 
                                 y = max(high, high[1]), 
                                 text = divergence_text_top,
                                 color = top_label_col,
                                 textcolor = neg_div_text_col,
                                 style = label.style_label_down
                                 ))
                                 
    if divergence_text_bottom != ""
        if showlast
            delete_old_pos_div_labels()
        array.push(pos_div_labels, 
                      label.new( x = bar_index, 
                                 y = min(low, low[1]), 
                                 text = divergence_text_bottom,
                                 color = bottom_label_col, 
                                 textcolor = pos_div_text_col,
                                 style = label.style_label_up
                                 ))

// Add TSA TDI SharkFin Alert
tdi_finUp = tdi_fastMA > 75 and tdi_fastMA > tdi_up
tdi_finDn = tdi_fastMA < 25 and tdi_fastMA < tdi_dn
tdi_alert = tdi_finUp or tdi_finDn

// Plot TSA TDI elements if showing TDI on chart
plot(show_tdi ? tdi_ma : na, "TDI Middle Band", color.yellow)
plot(show_tdi ? tdi_up : na, "TDI Upper Band", color.red)
plot(show_tdi ? tdi_dn : na, "TDI Lower Band", color.green)
plot(show_tdi ? tdi_mid : na, "TDI Mid Line", color.white)
plot(show_tdi ? tdi_fastMA : na, "TDI Fast MA", color.aqua, 2)

// STRATEGY EXECUTION SECTION

// Function to safely check if we can access historical data
safe_access(series, index) =>
    index >= 0 and index <= bar_index ? series[index] : na

// Safe TDI band checking for filter conditions
tdi_div_below_lower = false
tdi_div_above_upper = false

if div_start_bar_index > 0 and div_start_bar_index <= bar_index
    idx = bar_index - div_start_bar_index
    if idx >= 0 and idx <= bar_index
        tdi_div_below_lower := not na(tdi_value_at_div_start) and not na(safe_access(tdi_dn, idx)) and tdi_value_at_div_start < safe_access(tdi_dn, idx) and tdi_value_at_div_start < 25
                            
        tdi_div_above_upper := not na(tdi_value_at_div_start) and not na(safe_access(tdi_up, idx)) and tdi_value_at_div_start > safe_access(tdi_up, idx) and tdi_value_at_div_start > 75

// Modified filters: Removed TDI level confirmation
buy_filter = pos_reg_div_detected
sell_filter = neg_reg_div_detected

// NEW: Add candle confirmation logic
// Identify bullish and bearish candles
bullish_candle = close > open
bearish_candle = close < open

// Store signal states to track when a signal is active but waiting for candle confirmation
var bool buy_signal_active = false
var bool sell_signal_active = false

// Update signal states
if buy_filter
    buy_signal_active := true
if sell_filter
    sell_signal_active := true

// Final condition that requires both the signal and the candle confirmation
buy_condition = buy_signal_active and bullish_candle[1]  // Current candle bullish after signal detected
sell_condition = sell_signal_active and bearish_candle[1]  // Current candle bearish after signal detected

// Reset signal states after they trigger
if buy_condition
    buy_signal_active := false
if sell_condition
    sell_signal_active := false

// Function to convert pips to price for different instruments
pip_value() =>
    syminfo.mintick * (syminfo.type == "forex" ? 10 : 1)

// Execute strategy with pip-based SL/TP
if buy_condition
    strategy.entry("Long", strategy.long)
    if use_stop_loss
        sl_price = close - stop_loss_pips * pip_value()
        tp_price = use_take_profit ? close + take_profit_pips * pip_value() : na
        strategy.exit("SL/TP Long", "Long", stop=sl_price, limit=tp_price)
    else if use_take_profit
        tp_price = close + take_profit_pips * pip_value()
        strategy.exit("TP Long", "Long", limit=tp_price)

if sell_condition
    strategy.entry("Short", strategy.short)
    if use_stop_loss
        sl_price = close + stop_loss_pips * pip_value()
        tp_price = use_take_profit ? close - take_profit_pips * pip_value() : na
        strategy.exit("SL/TP Short", "Short", stop=sl_price, limit=tp_price)
    else if use_take_profit
        tp_price = close - take_profit_pips * pip_value()
        strategy.exit("TP Short", "Short", limit=tp_price)

// Plot buy and sell signals on chart
plotshape(buy_condition, title="Buy Signal", location=location.belowbar, color=color.green, style=shape.triangleup, size=size.small)
plotshape(sell_condition, title="Sell Signal", location=location.abovebar, color=color.red, style=shape.triangledown, size=size.small)

// Plot confirmation waiting states
plotshape(buy_signal_active and not buy_condition, title="Waiting for Buy Confirmation", location=location.belowbar, color=color.new(color.green, 70), style=shape.circle, size=size.tiny)
plotshape(sell_signal_active and not sell_condition, title="Waiting for Sell Confirmation", location=location.abovebar, color=color.new(color.red, 70), style=shape.circle, size=size.tiny)

// Alerts
//alertcondition(buy_condition, title='TSA TDI Buy Signal', message='TSA TDI Buy Signal')
//alertcondition(sell_condition, title='TSA TDI Sell Signal', message='TSA TDI Sell Signal')
