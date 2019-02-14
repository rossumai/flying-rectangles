Code used to export the dataset was (roughly):  
  
    def export_anonymized_dataset(config):
        for i, document in enumerate(dataset.documents):
            for field in filter_labels(document.field_extractions, ['text', 'table_body', 'table_header', 'table_footer',
                    'amount_total', 'amount_total_base', 'amount_total_tax',
                    'amount_rounding', 'amount_paid', 'amount_due',
                    'tax_detail_base', 'tax_detail_rate', 'tax_detail_tax', 'tax_detail_total',
                    'account_num', 'bank_num', 'iban', 'bic',
                    'const_sym', 'spec_sym', 'var_sym',
                    'invoice_id', 'order_id', 'customer_id',
                    'date_issue', 'date_uzp', 'date_due', 'terms',
                    'sender_ic', 'sender_dic', 'recipient_ic', 'recipient_dic',
                    'sender_name', 'sender_addrline', 'recipient_name', 'recipient_addrline',
                    'page_current', 'page_total',
                    'phone_num']):
                if field.fieldtype == 'text':
                    features = features_from_text(field.text, [100.0, 100000.0, 1000000000.0],
                                                  scale=20,
                                                  char_vocab=None)
                else:
                    features = None
                results.append([i, field.bbox, field.page, field.fieldtype, features])               
        import json
        with open('flying_rectangles.json', 'w') as outfile:
            json.dump(results, outfile)
      
      
    def features_from_text(text, values_scales, scale=20):
        try:
            xtextasval = float(text.replace(" ", "").replace("%", ""))
            xtextisval = 1.0
            assert np.isfinite(xtextasval)
        except:
            xtextasval = 0.0
            xtextisval = 0.0
        if xtextisval > 0.0:  # is actually a value
            xtextasval = [min(xtextasval / scale, 1.0) for scale in values_scales]
        else:
            xtextasval = [0.0] * len(values_scales)
    
        allfeats = base_text_features(text, scale=scale, features=['len', 'upper', 'lower', 'alpha', 'digit'])
        if len(text) <= 1:
            # just if we use the histograms for first two letters and last two letters, what to do in smaller
            text_to_handle = " " + text + " "
        else:
            text_to_handle = text
        begfeats = base_text_features(text_to_handle[0:2], scale=scale, features=['upper', 'lower', 'alpha', 'digit'])
        endfeats = base_text_features(text_to_handle[-2:0], scale=scale, features=['upper', 'lower', 'alpha', 'digit'])
    
        return allfeats + begfeats + endfeats + xtextasval + [xtextisval]
      
      
    def base_text_features(text, features=['len', 'upper', 'lower', 'alpha', 'digit'],
                           scale=20):
    
        def count_uppers(text):
            return sum([letter.isupper() for letter in text])
    
        def count_lowers(text):
            return sum([letter.islower() for letter in text])
    
        def count_alphas(text):
            return sum([letter.isalpha() for letter in text])
    
        def count_digits(text):
            return sum([letter.isdigit() for letter in text])
    
        use_cases = {
            'len': len,
            'upper': count_uppers,
            'lower': count_lowers,
            'alpha': count_alphas,
            'digit': count_digits,
        }
        repr = [use_cases[feature](text) for feature in features]
    
        if scale is not None:
            for i in range(len(repr)):
                repr[i] = min(repr[i] / scale, 1.0)
    
        return repr
    
  
Also we will try to upload it to http://academictorrents.com/  
